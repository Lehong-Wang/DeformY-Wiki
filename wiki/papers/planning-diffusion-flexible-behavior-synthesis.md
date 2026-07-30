---
title: "Planning with Diffusion for Flexible Behavior Synthesis"
slug: "planning-diffusion-flexible-behavior-synthesis"
arxiv: "2205.09991"
venue: "ICML 2022"
year: 2022
tags: [diffusion-model, planning, model-based-reinforcement-learning, trajectory-optimization, offline-rl, classifier-guidance, generative-planning, long-horizon-planning, control-as-inference, behavior-synthesis]
importance: 5
date_added: 2026-06-16
source_type: tex
s2_id: "3ebdd3db0dd91069fa0cd31cbf8308b60b1b565e"
tldr: "Diffuser is a trajectory-level diffusion model that fuses modeling and planning into one denoising process — sampling generates whole (state, action) trajectories on the data manifold, and lightweight cost/classifier guidance (or inpainting constraints) steers samples toward high-return or goal-satisfying plans, sidestepping the model-exploitation failures of trajectory optimization over a learned model."
contribution_type: [method, analysis]
datasets: [Maze2D, Multi2D, D4RL-locomotion, block-stacking]
code_url: "https://github.com/jannerm/diffuser"
cited_by: ["[[dynamic-manipulation-deformable-objects-3d-simulation]]"]
---

## Problem & Context

Planning with a learned model is the conceptually cleanest recipe for data-driven decision-making: learn an approximate forward dynamics model by supervised regression, then hand it to a classical trajectory optimizer. The trouble is that **this combination rarely works as advertised**. Powerful trajectory optimizers *exploit* the learned model — they find action sequences that look optimal under the model but are adversarial examples with respect to the true dynamics, so the resulting "plans" resemble exploits rather than behaviors. The field's pragmatic response had been twofold: (i) lean back toward model-free machinery (value functions, policy gradients) inside nominally model-based methods, and (ii) where online planning is kept, restrict it to weak, gradient-free optimizers — random shooting or the cross-entropy method (the [[deep-reinforcement-learning-handful-trials-using]] / PETS regime) — precisely to avoid the exploitation pathology.

Where the field stood: deep MBRL had explored a zoo of dynamics-model parameterizations (recurrent state-space models, VQ autoencoders, neural ODEs, normalizing flows, GNNs, Transformers) but nearly all assumed an **abstraction barrier** between model and planner — learning's only job was to approximate environment dynamics, after which any planner could be bolted on because the planner's form did not depend on the model's. This paper attacks that barrier directly. Its thesis: the diagnosis "learned models are ill-suited to standard trajectory optimization" should be answered not by a better learned dynamics model but by a model **designed in line with the planning problem it will be used in** — where action distributions matter as much as state dynamics, long-horizon accuracy matters more than single-step error, and the model's *plans* (not just its predictions) improve with data.

## Key idea

**Fold as much of the trajectory-optimization pipeline as possible into the generative model, so that sampling from the model and planning with it become nearly identical.** Concretely, train a single diffusion probabilistic model over *full trajectories of interleaved states and actions* on feasible/offline behavior data, $p_\theta(\boldsymbol\tau)$. Planning is then inference in a *perturbed* distribution

$$\tilde p_\theta(\boldsymbol\tau) \propto p_\theta(\boldsymbol\tau)\, h(\boldsymbol\tau),$$

where the perturbation $h(\boldsymbol\tau)$ encodes prior evidence (an observed start state), desired outcomes (a goal), or a general objective (reward/cost). Finding a $\boldsymbol\tau$ that is simultaneously high-likelihood under $p_\theta$ (a *physically realistic, in-distribution* trajectory) and high-value under $h$ is the probabilistic analogue of trajectory optimization. Because the dynamics information lives in $p_\theta$ and is **separated from the reward in $h$**, one trained diffusion model is reused across many tasks (and rewards unseen at training) in the same environment.

Two specific instantiations are RL counterparts of standard diffusion techniques:

1. **Reinforcement learning as classifier-guided sampling.** Via the control-as-inference graphical model, set $h(\boldsymbol\tau)=p(\mathcal{O}_{1:T}\mid\boldsymbol\tau)$ with $p(\mathcal{O}_t{=}1)=\exp(r(\mathbf s_t,\mathbf a_t))$. The reverse-diffusion transition is shifted by the gradient of a learned return predictor $\mathcal{J}_\phi$ — exactly Dhariwal & Nichol's classifier guidance, but the "classifier" is a *return/cost* function.
2. **Goal-conditioned RL as inpainting.** Constraints (start state, goal location) are treated like observed pixels in image inpainting: sample the unobserved trajectory entries to be consistent with the fixed conditioning entries.

The structural payoff for the DeformY arc: because every sampled plan is drawn from the *learned distribution of feasible behavior*, the planner stays **on the data manifold** by construction. This is a different defense against model exploitation than PETS's uncertainty-propagation — instead of penalizing the planner for leaving the trustworthy region, the generative prior makes off-manifold (adversarial) trajectories low-probability in the first place.

## Method

**Trajectory representation.** A plan is a 2-D array with one column per planning timestep, stacking states and actions:
$$\boldsymbol\tau = \begin{bmatrix} \mathbf s_0 & \mathbf s_1 & \cdots & \mathbf s_T \\ \mathbf a_0 & \mathbf a_1 & \cdots & \mathbf a_T \end{bmatrix}.$$
States and actions are predicted **jointly** — for prediction purposes actions are just extra state dimensions. This is the load-bearing design choice: the model is graded on the quality of the *controller* it induces, so action distributions are first-class, not an afterthought.

**Non-autoregressive, temporally local denoising.** Because decision-making is anti-causal (a present action conditions on *future* desired states, e.g. $p(\mathbf s_1\mid\mathbf s_0,\mathbf s_T)$), Diffuser cannot predict states autoregressively in time. It predicts *all* timesteps of a plan **concurrently**. The denoiser is built from repeated 1-D *temporal* convolutional residual blocks (a U-Net with 2-D spatial convs replaced by 1-D temporal convs), giving each denoising step a receptive field over only *nearby* timesteps (past and future). Local consistency, composed over many denoising steps, drives **global coherence** — this is the mechanism behind temporal compositionality. Being fully convolutional in the horizon dimension, the **planning horizon is set by the input noise size, not the architecture**, so plan length can change dynamically at test time.

**Training.** Standard DDPM $\epsilon$-prediction objective $\mathcal{L}(\theta)=\mathbb{E}_{i,\epsilon,\boldsymbol\tau^0}\lVert\epsilon-\epsilon_\theta(\boldsymbol\tau^i,i)\rVert^2$, reverse-process covariances on the cosine schedule (Nichol & Dhariwal). Two "times" coexist: diffusion step (superscript $i$) and planning step (subscript $t$).

**Guided planning (Algorithm 1, "Guided Diffusion Planning").** At control time, in a receding-horizon loop: observe state $\mathbf s$; initialize the plan as Gaussian noise $\boldsymbol\tau^N\sim\mathcal N(\mathbf 0,\mathbf I)$; for $i=N,\dots,1$ compute the reverse-transition mean $\mu\gets\mu_\theta(\boldsymbol\tau^i)$, then sample $\boldsymbol\tau^{i-1}\sim\mathcal N(\mu+\alpha\,\Sigma\,\nabla\mathcal J(\mu),\,\Sigma^i)$ (guidance shift) and **overwrite the first state $\boldsymbol\tau^{i-1}_{\mathbf s_0}\gets\mathbf s$** (inpainting the current state at every diffusion step); execute the first action $\boldsymbol\tau^0_{\mathbf a_0}$ and replan. The guidance gradient reduces exactly to $g=\nabla_{\mathbf s,\mathbf a}\sum_t r(\mathbf s_t,\mathbf a_t)\big|_\mu=\nabla\mathcal J(\mu)$ — the gradient of the planning objective. The return predictor $\mathcal J_\phi$ is trained separately on the same trajectory data (it reuses the first half of the diffusion U-Net).

**Inpainting for constraints.** For goal-reaching, $h(\boldsymbol\tau)$ is a Dirac delta on observed entries (start/goal) and uniform elsewhere; implemented by sampling the reverse process and replacing constrained entries with their conditioning values after each diffusion step. Reward maximization still uses inpainting for the start state. Multiple perturbations *compose* by adding their gradients — e.g. block stacking combines a final-state-match guide with an end-effector-contact constraint.

**Warm-started sampling for speed.** Plans are slow because generation is iterative. To speed open-loop replanning, run a few *forward* diffusion steps from the previous plan, then denoise back — reusing the prior plan as a warm start. Performance degrades only modestly as the per-step denoising budget is cut from 100 to as few as ~2 steps. Representative hyperparameters: guide scale $\alpha=0.1$ (most tasks); $N=20$ diffusion steps for locomotion, $N=100$ for block-stacking; planning horizon $T=100$ (locomotion), $128$ (block-stacking / large mazes).

## Experiment & Results

Three capabilities are tested: (1) long-horizon planning without reward shaping, (2) generalization to test-time goals unseen in training, (3) recovering a controller from heterogeneous-quality data; plus a runtime study.

**Long-horizon planning — Maze2D / Multi2D (D4RL), score normalized so 100 = reference expert.** Sparse reward (1 only at the goal); hundreds of steps to reach it, so credit assignment is hard. Diffuser conditions on start+goal by inpainting and rolls out the plan open-loop.

| Setting | MPPI (true dynamics) | CQL | IQL | **Diffuser** |
|---|---|---|---|---|
| Single-task average | 16.2 | 7.7 | 47.0 | **119.5** |
| Multi-task average (Multi2D) | 21.5 | — | 16.9 | **129.4** |

Diffuser exceeds 100 in *every* maze size (better than the reference expert), and — being a goal-conditioned planner by construction — loses **nothing** moving single-task → multi-task (no retraining, just change the conditioning goal), whereas IQL collapses (47.0 → 16.9). Crucially, **MPPI with ground-truth dynamics still scores only 16.2/21.5**: long-horizon sparse-reward planning is hard *even with a perfect model*, isolating the value of learned non-greedy planning.

**Test-time flexibility — block stacking (Kuka), score 100 = perfect stack, 0 = random.** One Diffuser trained on 10k PDDLStream demos, only $h(\boldsymbol\tau)$ changed per task.

| Task | BCQ | CQL | **Diffuser** |
|---|---|---|---|
| Unconditional Stacking | 0.0 | 24.4 | **58.7** |
| Conditional Stacking | 0.0 | 0.0 | **45.6** |
| Rearrangement | 0.0 | 0.0 | **58.9** |
| Average | 0.0 | 8.1 | **54.4** |

The conditional settings — which demand *flexible* behavior generation through novel states — are where model-free offline RL scores **0.0** while Diffuser composes goal-match + contact-constraint guides to succeed.

**Offline RL — D4RL locomotion (HalfCheetah / Hopper / Walker2d, 9 datasets), 150-seed mean.** Guided sampling (return guide) + inpainting (current state).

- Diffuser average **77.5**, essentially tied with the best offline methods (CQL 77.6, IQL 77.0, TT 78.9) and ahead of model-based MOReL (72.9), MBOP (47.8) and return-conditioning DT (74.7). On this single-task, well-studied regime Diffuser is *comparable*, not dominant — its edge is on the long-horizon / flexible tasks above.

**The diagnostic ablation (most load-bearing for DeformY).** The authors also tried using Diffuser *as a dynamics model* inside a conventional optimizer (MPPI), and it **"performed no better than random."** This is the paper's own evidence that Diffuser's effectiveness comes from the *coupled* modeling-and-planning design, **not** from improved open-loop predictive accuracy — plugging a generative trajectory model back into a classical planner reintroduces the exact exploitation failure the method was built to avoid.

**Runtime.** Warm-started sampling trades quality vs. budget gracefully; the denoising-step count per replan can be cut sharply (≈100 → single digits) with only a modest performance drop.

## Limitations

- **Slow generation.** Each plan is an iterative denoising sweep; naive open-loop replanning regenerates a plan every step. Warm-starting mitigates but does not eliminate the cost — real-time control is not demonstrated.
- **Not best on standard single-task offline RL.** On D4RL locomotion Diffuser only matches (does not beat) the strongest offline methods; its advantages are specific to long-horizon and test-time-flexible settings.
- **Known reward / cost for guidance.** Like other planning methods it needs a return predictor $\mathcal J_\phi$ (trained on the data) or an explicit cost to guide sampling; the guidance is a *soft* approximation valid when $p(\mathcal O_{1:T}\mid\boldsymbol\tau^i)$ is sufficiently smooth.
- **Guide-scale sensitivity.** The guidance scale $\alpha$ must be tuned (and lowered for short horizons), indicating the score+guidance balance is not automatic.
- **Offline / demonstration data assumed.** The model is trained on collected trajectories; it is a behavior-synthesis-from-data method, not an online explorer.
- **Evaluated on low-to-moderate-DoF simulated control** (point mazes, MuJoCo locomotion, Kuka block stacking); scaling to high-DoF, contact-rich, or deformable systems is left open.

## Open questions

- How far does trajectory-level diffusion planning scale to **high-DoF, contact-rich, or deformable** dynamics (rope/cloth), where staying on the data manifold could be an especially strong defense but trajectory dimensionality explodes? (This is precisely the lever the DLO line — e.g. [[dynamic-manipulation-deformable-objects-3d-simulation]] — pulls.)
- Can the slow iterative sampling be cut to **closed-loop control rates** (consistency/distillation, few-step samplers) without losing the guidance budget that makes planning work?
- Is the on-manifold generative prior a *complete* substitute for explicit uncertainty handling, or do the [[deep-reinforcement-learning-handful-trials-using]] (PETS) uncertainty-propagation defense and Diffuser's manifold-constraint defense compose into something stronger than either alone?
- How well-calibrated is the separation between "dynamics in $p_\theta$" and "reward in $h$" when the offline data poorly covers the rewarded region — does guidance then push samples off-distribution after all?

## My take

Diffuser is the **founding paper of planning-as-diffusion** and the cleanest articulation of a structural idea: if a learned model will be used for planning, design the model so that sampling *is* planning, and the planner can no longer exploit it because every candidate trajectory is drawn from the learned distribution of feasible behavior. The reframing of classifier guidance as cost-guided planning and of inpainting as goal conditioning is elegant and has proven enormously generative — it is the direct ancestor of essentially every "diffusion for control / decision diffuser / diffusion policy for planning" paper that followed, including the DLO whipping line in this wiki.

For the DeformY arc the paper is doubly load-bearing. First, it is the **principal's chosen cost-guided trajectory-generation planner**: a diffusion model over feasible rope behavior, steered by a tip-targeting cost gradient at sampling time, is exactly the planning-as-diffusion recipe applied to deformables — and [[dynamic-manipulation-deformable-objects-3d-simulation]] (DIDP) is essentially that idea instantiated for 3D rope whipping (its PITA test-time guidance is a physics-loss specialization of Diffuser's cost guidance). Second, it is the **structural defense against model exploitation** that pairs with PETS: the two papers stake out the two principled routes to robust planning over a learned model — PETS propagates *uncertainty* so a sampling planner ignores the untrustworthy far future, while Diffuser constrains plans to the *data manifold* so adversarial trajectories are simply low-probability. The self-reported MPPI-on-Diffuser-is-random result is the sharpest evidence that the coupling, not the predictive accuracy, is the source of the gain — a useful warning for any follow-on that hopes to reuse a generative trajectory model inside a classical optimizer. The honest caveat is that Diffuser only *ties* the best offline-RL methods on standard single-task benchmarks and is slow; its real value is long-horizon, multi-task, test-time-flexible behavior synthesis, which is precisely the regime that goal-conditioned dynamic DLO manipulation lives in.

## Related

- [[model-based-planning-for-manipulation]]
**Foundations used**

- [[denoising-diffusion-probabilistic-models]] — the generative backbone; Diffuser is a DDPM whose data are full trajectories, with $\epsilon$-prediction training and a cosine reverse-covariance schedule
- [[trajectory-optimization]] — the classical problem Diffuser reframes; planning becomes inference in a reward-perturbed trajectory distribution, the probabilistic analogue of $\arg\max_{\mathbf a_{0:T}}\sum_t r(\mathbf s_t,\mathbf a_t)$
- [[model-based-reinforcement-learning]] — the host paradigm; Diffuser is a new class of deep-MBRL planner that breaks the model/planner abstraction barrier
- [[diffusion-policy]] — sibling use of diffusion for control; diffusion policies model the action distribution given an observation, whereas Diffuser models whole (state, action) trajectories for planning (Diffuser predates and motivates much of this family)

**Concepts introduced**

- [[planning-as-diffusion]] — fold trajectory optimization into a trajectory-level diffusion model so that sampling and planning coincide, with cost/classifier guidance and inpainting as the planning operators (this paper coins and defines the formulation)

**Methods introduced**

- [[diffuser-guided-diffusion-planning]] — the named, reusable algorithm: a temporal-convolution trajectory diffusion model trained on feasible behavior, planned by guided reverse diffusion (return-gradient guidance + start/goal inpainting) in a receding-horizon loop

**People**

- [[michael-janner]] — first author

**Similar methods**

- [[mmp-motion-manifold-primitives-parametric-curve]] — MMP++: the autoencoder-manifold route to the same formulation — plan by generating from a learned model of the trajectory distribution — with an explicit latent density + constrained latent optimization in place of diffusion + guidance
- [[yilun-du]] — second author
- [[sergey-levine]] — senior author

**Cross-paper relations**

- [[dynamic-manipulation-deformable-objects-3d-simulation]] — `similar_method_to`: DIDP applies guided diffusion over full trajectories to 3D rope whipping; it is the deformable-object instantiation of Diffuser's planning-as-diffusion, with PITA as a differentiable-physics specialization of Diffuser's cost/classifier guidance.
- [[deep-reinforcement-learning-handful-trials-using]] — `same_problem_as`: both target robust planning over an imperfect *learned* model and answer the same model-exploitation failure with two different structural defenses — PETS propagates ensemble uncertainty so a gradient-free sampler (CEM-MPC) ignores the untrustworthy far future; Diffuser keeps plans on the data manifold via a generative trajectory prior. Diffuser's intro explicitly names the CEM/random-shooting (PETS-style) regime as the workaround it supersedes.

**Important referenced work** (evidence-text only — full citations live in `graph/citations.jsonl`)

- PETS (Chua et al. 2018) — [[deep-reinforcement-learning-handful-trials-using]]; the gradient-free CEM-MPC-over-a-learned-model baseline Diffuser positions itself against (bibliographic `cites` row added).
- DDPM (Ho et al. 2020), Improved DDPM (Nichol & Dhariwal 2021), Diffusion Models Beat GANs (Dhariwal & Nichol 2021) — the diffusion machinery and classifier-guidance technique Diffuser adapts to planning.
- Decision Transformer (Chen et al. 2021), Trajectory Transformer (Janner et al. 2021) — sequence-modeling approaches to offline RL compared against in the locomotion table.
- D4RL (Fu et al. 2020) — the Maze2D / locomotion benchmark suite.
