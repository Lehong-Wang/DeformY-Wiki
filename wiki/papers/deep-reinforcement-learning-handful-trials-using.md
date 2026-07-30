---
title: "Deep Reinforcement Learning in a Handful of Trials using Probabilistic Dynamics Models"
slug: "deep-reinforcement-learning-handful-trials-using"
arxiv: "1805.12114"
venue: "NeurIPS 2018"
year: 2018
tags: [model-based-reinforcement-learning, probabilistic-ensemble, trajectory-sampling, uncertainty-aware-dynamics, aleatoric-uncertainty, epistemic-uncertainty, model-predictive-control, cross-entropy-method, sample-efficiency, continuous-control]
importance: 5
date_added: 2026-06-16
source_type: tex
s2_id: "56136aa0b2c347cbcf3d50821f310c4253155026"
tldr: "PETS combines a probabilistic ensemble of bootstrapped neural dynamics models (separating aleatoric from epistemic uncertainty) with trajectory-sampling uncertainty propagation and CEM-based MPC planning, matching model-free asymptotic returns on MuJoCo control with 8-125x fewer samples."
contribution_type: [method, analysis]
datasets: []
code_url: "https://github.com/kchua/handful-of-trials"
cited_by: ["[[self-curriculum-model-based-reinforcement-learning]]", "[[learning-adapt-dynamic-real-world-environments]]", "[[rapid-adaptation-particle-dynamics-generalized-deformable]]", "[[planning-diffusion-flexible-behavior-synthesis]]"]
---

## Problem & Context

Model-based reinforcement learning (MBRL) is attractive because a learned, reward-independent dynamics model is sample-efficient and reusable across tasks in the same environment, and it can absorb every advance in deep supervised learning. But on standard continuous-control benchmarks MBRL methods had consistently *learned fast yet converged worse* than model-free methods — the asymptotic-performance gap. This paper attacks that gap directly: can a *purely* model-based deep-RL algorithm (no parameterized policy, only a dynamics model + planner) match the asymptotic returns of the best model-free methods while keeping MBRL's order-of-magnitude sample advantage?

Where the field stood: Bayesian nonparametric models — Gaussian processes — were the MBRL model of choice in low-dimensional, data-scarce regimes (PILCO, Deisenroth & Rasmussen), but GPs scale poorly with dimension and their smooth (squared-exponential) kernels are ill-suited to the contact discontinuities typical of robotics. Neural-network dynamics models scale to high dimensions and large data and represent non-smooth dynamics, but they **overfit in the low-data regime** and make poor long-horizon predictions — which is exactly the regime MBRL starts in. Prior NN-MBRL therefore largely used *deterministic* models and underperformed model-free RL (Nagabandi et al. 2017's MBMF) or fell back on time-varying linear models (guided policy search). Uncertainty-aware deep nets (Bayesian NNs, MC-dropout, deep ensembles, α-divergence) had been explored, including in RL (Gal et al.'s Deep-PILCO, Depeweg et al.), but no work had combined the right uncertainty components into a deep-MBRL framework that *reached model-free asymptotic performance* on challenging benchmarks. The paper's thesis: the missing ingredient is the *correct, empirically-validated* treatment of uncertainty in both model learning and planning.

## Key idea

**Properly incorporating uncertainty into a high-capacity neural dynamics model — and propagating that uncertainty through planning — closes the MBRL-vs-model-free asymptotic gap.** Three observations drive the design:

1. **Model capacity matters**, so use neural networks (not GPs) to represent complex, discontinuous dynamics at scale.
2. **Two distinct uncertainties must both be modeled.** *Aleatoric* uncertainty (inherent system stochasticity / observation+process noise) is captured by a **probabilistic** network that outputs the parameters of a distribution (e.g. a Gaussian mean and input-dependent variance) rather than a point. *Epistemic* uncertainty (subjective, from finite data — the part that *vanishes with more data* and mitigates overfitting) is captured by an **ensemble** of bootstrapped models. Combining the two gives the **probabilistic ensemble (PE)** — the only one of {D, P, DE, GP, PE} that captures both.
3. **Uncertainty must be propagated, not collapsed.** A **trajectory-sampling (TS)** particle scheme rolls many particles through the PE model, each particle bound to a bootstrap, so the planner sees the full (possibly multimodal) trajectory distribution. The TS∞ variant (each particle keeps its bootstrap for the whole horizon) makes aleatoric and epistemic uncertainty *separable* at plan time.

The full algorithm — **PETS (Probabilistic Ensembles with Trajectory Sampling)** — plans with sampling-based **MPC** using the **cross-entropy method (CEM)** to optimize action sequences, executing only the first action and replanning each step. Notably, PETS learns *only* a dynamics model — no policy network — and still matches model-free asymptotes.

## Method

PETS instantiates the standard MBRL loop over a Markov decision process ([[markov-decision-process]]) — fit a forward dynamics model $\tilde{f}$ to data $\mathcal{D}=\{(s_n,a_n),s_{n+1}\}$, plan with it, execute, append data, refit — but makes three specific, empirically-justified design choices.

**1. Dynamics model — probabilistic ensemble (PE).** The model is an ensemble of $B$ bootstrapped *probabilistic* neural networks, realizing the [[probabilistic-ensemble]] foundation:

- Each network is **probabilistic**: it outputs a conditional Gaussian $\tilde{f}_\theta = \mathcal{N}\!\big(\mu_\theta(s_t,a_t),\,\Sigma_\theta(s_t,a_t)\big)$ with input-dependent (heteroscedastic) diagonal covariance, trained by the **negative-log-likelihood** loss $\text{loss}_P(\theta)=-\sum_n \log \tilde{f}_\theta(s_{n+1}\mid s_n,a_n)$. This captures **aleatoric** uncertainty. (A deterministic net D is the special case of a delta-distribution / MSE loss, whose "variance" carries no usable uncertainty.)
- The model is an **ensemble** of $B$ networks, each trained on its own bootstrap resample $\mathcal{D}_b$ (sample-with-replacement of $\mathcal{D}$). Bootstrap *disagreement* far from the data captures **epistemic** uncertainty — the cheap, hyperparameter-free alternative to full Bayesian NN inference. The authors found **$B=5$** sufficient for every task.
- The {aleatoric, epistemic} coverage of the model variants is the paper's organizing table: D (neither), P (aleatoric only), DE (epistemic only), GP (epistemic + homoscedastic aleatoric), and **PE (both)**. The measured model ranking is **PE > P > DE > D**.
- An under-appreciated detail: a probabilistic net's predicted variance is *arbitrary* on out-of-distribution inputs and can wreck planning; PETS bounds the output variance (appendix) to keep MPC stable.

**2. Uncertainty propagation — trajectory sampling (TS).** To evaluate a candidate action sequence under the PE model, PETS Monte-Carlo-propagates $P$ state particles ($s^p_{t=0}=s_0$), each stepped by $s^p_{t+1}\sim \tilde{f}_{\theta_{b(p,t)}}(s^p_t,a_t)$ under a per-particle bootstrap index $b(p,t)$. Two variants:

- **TS1** re-samples each particle's bootstrap *every* timestep — particles draw from the approximate marginal posterior of plausible dynamics, softly limiting trajectory multimodality.
- **TS∞** fixes each particle's bootstrap for the *entire* trial, respecting the time-invariance of the true dynamics. Its key property: **aleatoric and epistemic state variance become separable** — aleatoric is the average within-bootstrap particle variance; epistemic is the variance of the per-bootstrap particle means. Only epistemic uncertainty is the "learnable" kind worth exploring; conflating the two would mislead an exploration rule into chasing irreducible noise.

Baseline propagation methods compared (to isolate TS's value): Expectation (E, mean-only, no uncertainty), Moment Matching (MM, recast to Gaussian each step), and Distribution Sampling (DS, moment-match over bootstraps only). PETS uses **$P=20$** particles.

**3. Planning — CEM-based MPC.** Rather than learn a policy, PETS plans with **model predictive control** ([[model-predictive-control]]): at each step it optimizes an action sequence over horizon $H$, $\arg\max_{a_{t:t+H}}\sum_{\tau}\mathbb{E}_{\tilde f}[r(s_\tau,a_\tau)]$, evaluated as the per-particle average reward; it executes only the first action and replans. MPC is chosen for implementation simplicity, no test-time gradients, and no need to fix the task horizon. The inner sequence optimizer is the **cross-entropy method** ([[cross-entropy-method]]) — a sampling-based [[trajectory-optimization]] method that iteratively refits a sampling distribution to the elite (highest-reward) action samples — which the authors show *significantly* outperforms plain random shooting on the ground-truth half-cheetah (5 CEM iterations × 500 candidates ≈ 2500 samples vs. 2500 random samples). Probabilistic propagation also makes MPC robust to an over-long horizon: particle separation makes the planner effectively ignore the unpredictable far future, whereas deterministic propagation compounds model bias.

**Full PETS loop (Algorithm 1):** initialize $\mathcal{D}$ from one random-controller trial; for each trial $k=1..K$: train the PE model on $\mathcal{D}$; for each timestep, run CEM (sampling action sequences, propagating particles via TS, scoring by mean particle reward, updating the CEM distribution), execute the first optimized action, and append the observed transition to $\mathcal{D}$.

## Experiment & Results

**Tasks (MuJoCo).** Cartpole, 7-DoF Pusher, 7-DoF Reacher (PR2), and Half-cheetah — spanning low-dimensional to high-dimensional, contact-rich dynamics. One timestep = 0.01 s (0.02 s for Cartpole). Curves are the max-reward-so-far averaged over 10 runs.

**Comparison to prior RL (asymptote + sample efficiency).** Baselines: model-free PPO, DDPG, SAC; model-based deterministic MBMF (Nagabandi et al. 2017, which is essentially PETS's D-E variant); and GP-based MBRL (GP-E, GP-DS, GP-MM).

- **PETS reaches PPO-level asymptotic performance on all four tasks in under 100 trials / 100K timesteps**, far faster than any model-free baseline.
- On **half-cheetah**, PETS reaches SAC/PPO-level returns with **~8× fewer samples than SAC and ~125× fewer than PPO** — the headline number.
- PETS **substantially exceeds the prior deterministic MBRL** (Nagabandi 2017 = the D-E ablation), quantifying the value of uncertainty.
- On the low-dimensional **Cartpole**, the GP baseline GP-MM slightly beats PETS (GPs excel when the problem is small), but GP-MM scales **cubically in horizon and quadratically in state dimension**, so it was infeasible on the higher-dimensional tasks.

**Ablation — model class × propagation method.** Across all tasks (except cartpole), the **probabilistic ensemble (PE) wins regardless of propagation method**; single-uncertainty-type models (P, DE) are close seconds; the deterministic D is worst — confirming the ranking **PE > P > DE > D**. The decisive lever is the **model and the use of uncertainty at *learning* time**; the choice of propagation method (TS vs MM vs DS vs E) yields only minor differences *given* a good model — though MM, competitive in low dimensions (consistent with Gal et al.), is unreliable in high dimensions (half-cheetah). The paper's framing: every individual component (ensembles, distribution-outputting nets, MPC for MBRL) existed before; the *systematic empirical determination of the right combination* is the contribution.

**Additional analyses (appendix).** CEM > random shooting on ground-truth dynamics; MPC-horizon sensitivity (probabilistic propagation is robust to over-long horizons, deterministic is not); and behavior under varying system stochasticity (where separating aleatoric/epistemic matters).

## Limitations

- **Exploration left on the table.** PETS *separates* epistemic uncertainty (via TS∞) but does not yet *use* it to direct exploration — the authors explicitly defer epistemic-uncertainty-driven exploration to future work, despite it being a natural payoff of the decomposition.
- **Planning, not policy — replanning cost.** PETS runs CEM-MPC at every timestep with no amortized policy, so test-time planning is expensive; the authors note adding a policy to amortize planning is desirable but their attempts to backprop policy gradients through the uncertainty-aware model *did not yield an effective algorithm* (suspected chaotic policy gradients).
- **Residual model-based sub-optimality.** Matching model-free asymptotes is demonstrated on these benchmarks, but MBRL's general tendency toward sub-optimal convergence is bridged "in part," not eliminated.
- **Reward/cost must be known.** Like other MPC-based MBRL, PETS assumes a known reward function to score sampled trajectories at plan time.
- **OOD variance pathology.** Probabilistic nets emit arbitrary variance off-distribution; PETS needs an explicit variance-bounding fix for stable planning — a symptom of the underlying calibration problem.
- **Ensemble + particle cost.** $B{=}5$ networks × $P{=}20$ particles × CEM iterations per step is a non-trivial compute and memory footprint relative to a single deterministic model.

## Open questions

- Can the separated **epistemic** signal be turned into a principled exploration bonus (the deferred contribution), and how much sample efficiency does that buy on hard-exploration tasks?
- Can a **policy be distilled / learned** on top of PETS to amortize the per-step CEM cost without losing the uncertainty benefits — perhaps via the chaotic-policy-gradient analyses the authors cite as a path to a "policy-based PETS"?
- How far does the PE+TS recipe transfer to **high-dimensional, contact-rich deformable-object dynamics** (rope/cloth), where a single accurate global model is hopeless and uncertainty-aware short-horizon planning could shine? (This is exactly the lever the DLO sampling-MPC lines below pull.)
- Is bootstrapped-ensemble disagreement a *well-calibrated* epistemic estimate, or does it under/over-cover far from data in ways that bias long-horizon plans?

## My take

This is the **canonical probabilistic-ensemble + sampling-planning backbone** of modern MBRL — the paper that made "neural-net MBRL can match model-free asymptotic performance at a fraction of the samples" the default expectation rather than a hope. Its real contribution is *epistemological*: a careful, ablation-driven demonstration that the gap was an **uncertainty-handling** problem, not a capacity problem, and that the fix factors cleanly into (i) a probabilistic ensemble that separates aleatoric from epistemic uncertainty and (ii) sampling-based uncertainty propagation feeding CEM-MPC. None of the parts was new; the *validated combination* was, and it has been the reference MBRL recipe ever since (PETS is the named baseline in essentially every MBRL paper that follows).

For the DeformY arc this paper is load-bearing as the **principal's chosen primary defense against model exploitation**: when you plan with a *learned* model, a deterministic planner will happily exploit the model's errors; PETS's answer — propagate uncertainty with an ensemble and let particle separation make the planner ignore the unpredictable — is precisely the mechanism that keeps sampling-MPC honest on an imperfect learned rope model. The connection to the rope/DLO lines is structural and direct: the **CEM-MPC-over-a-learned-model** planner that [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] runs as its gradient-free planning baseline *is* PETS-style sampling-MPC; [[wiggle-go-system-identification-zero-shot]]'s CMA-ES trajectory optimization over an identified rope model is the same sampling-planning-over-a-model pattern with a different sampler; [[self-curriculum-model-based-reinforcement-learning]] builds its DLO controller on an *ensemble* dynamics model and even uses PETS-style ensemble look-ahead inside its curriculum; and [[learning-adapt-dynamic-real-world-environments]] (Nagabandi/Finn) is the sibling deep-MBRL-with-MPC method that explicitly names PETS-style probabilistic ensembles as the orthogonal, complementary uncertainty mechanism its fixed-variance model lacks. The cleanest DeformY lever is the third open question: a PE forward-dynamics model of a rope (on a Cosserat-grade simulator) planned with TS+CEM-MPC, using epistemic disagreement to avoid exploiting the model where it is least trustworthy.

## Related

- [[model-based-planning-for-manipulation]]
**Foundations used**

- [[probabilistic-ensemble]] — the dynamics model: an ensemble of bootstrapped probabilistic NNs capturing both aleatoric and epistemic uncertainty (this paper is the canonical source of the "PE / PETS dynamics model")
- [[model-based-reinforcement-learning]] — the host paradigm; PETS is the algorithm that closed its asymptotic gap with model-free RL
- [[model-predictive-control]] — receding-horizon planning: optimize an action sequence, execute the first action, replan each step
- [[cross-entropy-method]] — the sampling-based inner optimizer for the MPC action search (beats random shooting)
- [[trajectory-optimization]] — sampling-based action-sequence optimization is the trajectory-optimization layer of the planner
- [[markov-decision-process]] — the control formalism for the learned forward dynamics

**Methods introduced**

- [[pets-probabilistic-ensembles-trajectory-sampling]] — the named, reusable MBRL algorithm: PE dynamics + trajectory-sampling propagation + CEM-MPC planning

**Concepts introduced**

- [[trajectory-sampling-uncertainty-propagation]] — the particle-based propagation scheme (TS1 / TS∞) that rolls bootstrap-bound particles through an ensemble dynamics model and makes aleatoric vs. epistemic uncertainty separable for planning

**People**

- [[sergey-levine]] — senior author
- [[kurtland-chua]] — first author

**Cited-by in this wiki** (papers that cite PETS)

- [[self-curriculum-model-based-reinforcement-learning]] — ensemble-dynamics MBRL for DLO shape control; cites PETS as MBRL backbone
- [[learning-adapt-dynamic-real-world-environments]] — deep MBRL + MPC; names PETS-style probabilistic ensembles as the complementary uncertainty mechanism
- [[rapid-adaptation-particle-dynamics-generalized-deformable]] — particle-dynamics MBRL for deformable mobile manipulation; cites PETS
- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser; cites PETS as the gradient-free CEM-MPC-over-a-learned-model regime it supersedes with on-manifold generative planning (`same_problem_as`)
