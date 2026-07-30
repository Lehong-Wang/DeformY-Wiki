---
title: "Learning to Adapt in Dynamic, Real-World Environments Through Meta-Reinforcement Learning"
slug: "learning-adapt-dynamic-real-world-environments"
arxiv: "1803.11347"
venue: "ICLR 2019"
year: 2019
tags: [meta-reinforcement-learning, model-based-reinforcement-learning, online-adaptation, dynamics-model, MAML, recurrent-model, MPC, sim-to-real, legged-robot, continuous-control]
importance: 4
date_added: 2026-06-16
source_type: tex
s2_id: "944bd3b472c8a30163bbfc1b5cbab8545693c3e0"
tldr: "Meta-trains a dynamics-model prior whose update rule rapidly re-fits the model from the last M timesteps to predict the next K, giving model-based RL agents (GrBAL/ReBAL) fast online adaptation to crippled limbs, new terrains, and payloads — demonstrated on the first meta-RL real robot."
contribution_type: [method, application]
datasets: []
code_url: "https://sites.google.com/berkeley.edu/metaadaptivecontrol"
cited_by: ["[[rma-rapid-motor-adaptation-legged-robots]]"]
---

## Problem & Context

Reinforcement learning policies — model-free or model-based — are typically trained to master a fixed setting and then deployed, but the real world perturbs that setting constantly: a robot loses a leg, the terrain changes, a payload is added, pose estimation drifts. A policy specialized to nominal dynamics fails under these shifts, and training a separate policy per situation is infeasible. Humans, by contrast, re-adapt within seconds (learning to walk on crutches, lifting an unexpectedly heavy object). The paper's question: can an agent *learn how to adapt online* so that, after a dynamics change, it recovers in a handful of timesteps?

Where the field stood: model-free RL needs enormous interaction and produces brittle specialists; model-based RL ([[model-based-reinforcement-learning]]) is more sample-efficient but is bottlenecked on acquiring a single **globally accurate** dynamics model — and even a perfect global model cannot capture dynamics that change as a function of unobservable environmental factors. Classical adaptive control ([[system-identification]]) adapts online but does not scale to high-dimensional nonlinear systems. Prior meta-RL ([[meta-learning]]) was almost entirely model-free, costing *more* meta-training samples than ordinary RL and operating at the **episodic** level (a "task" = a whole trajectory with a different reward/environment), which precludes real-robot use and within-episode adaptation. The one prior model-based meta-RL method (Sæmundsson et al. 2018) used GPs and episodic adaptation on cart-pole/pendulum. This paper reframes the problem so a high-capacity neural dynamics model can adapt *within* an episode, *at every timestep*, and trains it explicitly for that.

## Key idea

**Meta-learn a dynamics-model prior such that, combined with recent data, it adapts to the local context in one cheap update.** Two moves do the work:

1. **Timestep-as-task (online, non-episodic).** Instead of treating each trajectory as a task, treat every length-$(M{+}K)$ trajectory *segment* as a task: use the past $M$ timesteps to adapt the model and require it to predict the next $K$ timesteps well. Tasks are thus constructed automatically from experience — no hand-designed task distribution — under a mild "locally consistent environment" assumption (dynamics are constant over a short window).
2. **Train the prior *for* adaptability.** Optimize a meta-objective over $(\theta, \psi)$: the model prior $\theta$ and an update function $u_\psi$ are jointly trained so that adapting $\theta \to \theta' = u_\psi(\tau(t{-}M,t{-}1), \theta)$ on the last $M$ points minimizes next-$K$-step prediction loss. The adapted model feeds an MPC controller. Because the model adapts online, it need not be accurate everywhere a priori — it only has to be *quickly fixable*.

Two instantiations of the update function: **GrBAL** (gradient-based, a MAML inner step) and **ReBAL** (recurrence-based, a learned recurrent update of a hidden state).

## Method

**Setup.** Markov decision process ([[markov-decision-process]]) with a learned stochastic forward dynamics model $\hat p_\theta(\mathbf{s}'\mid\mathbf{s},\mathbf{a})$ — a neural net (3 hidden layers × 512, ReLU) outputting a Gaussian mean with fixed variance, so MLE reduces to MSE. A distribution $\rho(\mathcal{E})$ over *environments* shares state/action spaces but differs in dynamics $p_\mathcal{E}$.

**Meta-objective.** Optimize
$$\min_{\theta,\psi}\ \mathbb{E}_{\tau_\mathcal{E}(t-M,t+K)\sim\mathcal{D}}\big[\mathcal{L}(\tau_\mathcal{E}(t,t+K),\theta'_\mathcal{E})\big]\quad\text{s.t.}\quad \theta'_\mathcal{E}=u_\psi(\tau_\mathcal{E}(t-M,t-1),\theta),$$
with $\mathcal{L}$ the negative log-likelihood (mean over the $K$ future steps). The past $M$ points adapt $\theta$; the loss of the adapted $\theta'$ is scored on the future $K$ points — so training directly rewards "adapt-from-$M$, predict-the-next-$K$." $M$ and $K$ are hyperparameters (set $K{=}M$; the appendix shows performance is robust to their value).

**GrBAL (Gradient-Based Adaptive Learner).** The update rule is a MAML-style gradient step on the inner likelihood:
$$\theta'_\mathcal{E}=\theta_\mathcal{E}+\psi\,\nabla_\theta\frac{1}{M}\sum_{m=t-M}^{t-1}\log\hat p_{\theta_\mathcal{E}}(\mathbf{s}_{m+1}\mid\mathbf{s}_m,\mathbf{a}_m),$$
with $\psi$ a (learnable or fixed) inner learning rate; a single inner gradient step is used. The outer loop backpropagates the meta-loss through this inner step to update $\theta$ (and $\psi$). This is the online, self-constructed-task variant of MAML — unlike supervised MAML, the task distribution is built from temporal segments rather than hand-designed.

**ReBAL (Recurrence-Based Adaptive Learner).** $u_\psi$ is the recurrent cell's hidden-state update; $\psi$ are the recurrent weights and $\theta$ the rest of the network plus hidden state. The model *learns its own* update rule via its gating, rather than being told to use gradient descent.

**Online adaptation + control (test time).** At every timestep: adapt $\theta_* \to \theta_*' = u_{\psi_*}(\mathcal{D}(t{-}M,t{-}1),\theta_*)$, pass the adapted model to a sampling-based MPC controller ([[model-predictive-control]]) — **MPPI** (model-predictive path integral) in simulation, **random shooting** on the real robot — execute the first action, append the transition, **reset to $\theta_*$**, and repeat. The planning horizon $H$ is kept shorter than the adaptation horizon $K$ because the adapted model is only locally valid. MPC re-planning prevents error accumulation and lets the model improve again next step. The same online-adaptation rollout procedure is also used during meta-training to provide on-policy data.

## Experiment & Results

**Questions probed:** does adaptation actually change the model; does it enable fast adaptation in- and out-of-distribution; how does it compare to baselines; GrBAL vs ReBAL; sample efficiency vs model-free meta-RL; does it work on a real robot.

**Simulated environments (MuJoCo).** Half-cheetah with a randomly **disabled joint**, **sloped terrain**, and a **pier** of floating blocks with varying damping/friction; **Ant** with a randomly **crippled leg**. Each is meta-trained over a range of malfunctions/terrains and tested on *unseen* ones, plus mid-rollout transitions (e.g. switching which joint is disabled) to test fast adaptation.

**Baselines:** model-free RL (TRPO), model-free meta-RL (MAML-RL), plain model-based RL (MB), and MB with **dynamic evaluation** (MB+DE) — i.e. test-time gradient adaptation of a model that was *not* trained to adapt. All model-based methods share architecture, bootstrapping, and planner. An **MB oracle** trained with unlimited data from the single test environment is also included; returns are normalized so the oracle = 1.

**Findings.**
- **Adaptation changes the model:** the post-update model's $K$-step prediction-error histogram is shifted lower than the pre-update model's across all tasks — the $M$-step inner update genuinely reduces next-$K$-step error.
- **Sample efficiency:** GrBAL/ReBAL meta-train on the equivalent of **1.5–3 hours** of real-world experience — roughly **1000× less data** than TRPO/MAML-RL (trained to convergence on ~2 days of experience) — yet match or beat the model-free agent and surpass non-meta model-based methods. On HC-disabled they nearly match MAML-RL's asymptote; on Ant-crippled they trail it asymptotically (the known model-based sub-optimality) but with ~1000× less data.
- **Test-time adaptation & generalization:** GrBAL/ReBAL beat all baselines in every fast-adaptation/generalization setting. MB+DE generalizes better than MB but its *slow* adaptation lags MB where speed matters — showing the benefit of explicitly *training for* adaptability, not just adapting at test time. On the HC pier and Ant fast-adaptation, the method **beats the MB oracle**, because no amount of data makes a static model robust to mid-rollout disturbances. **GrBAL ≥ ReBAL** overall (ReBAL helps where longer input sequences better reveal the setting).
- **Real robot (VelociRoACH 6-legged millirobot).** 24-D state, 2-D action (per-side leg velocity setpoints streamed at 10 Hz), motion-capture room. Meta-trained on ~30 min each of **carpet, styrofoam, turf** (random policy). On training terrains, GrBAL and MB follow trajectories comparably (adaptation neither needed nor harmful — Table reports per-shape costs for left-turn/straight/zig-zag/figure-8). On **unseen** conditions — missing leg, novel terrains/slopes, pose miscalibration, pulling a payload — **GrBAL substantially outperforms MB and MB+DE**, adapting online to all four. The paper states this is, to the authors' knowledge, the **first meta-RL algorithm deployed on a real robotic system**.

## Limitations

- **Locally-consistent-environment assumption.** Adaptation assumes dynamics are constant over the length-$(M{+}K)$ window; abrupt changes *within* a window violate it (rarely, given sub-second adaptation, but it is an assumption).
- **Model-based asymptotic sub-optimality.** Final returns can trail model-free meta-RL (e.g. Ant-crippled) — an inherent model-based-RL limitation the authors flag.
- **Fixed-variance Gaussian model, no model uncertainty.** The dynamics model uses fixed variance; the authors note incorporating ensemble/uncertainty (e.g. PETS-style [[probabilistic-ensemble]]) is orthogonal and complementary but not done here.
- **Hyperparameters $M,K$** are pre-specified per agent; though robust in the sensitivity study, the optimal values depend on state informativeness and timestep duration.
- **MPC/planner dependence.** Control quality is tied to the sampling-based MPC (MPPI / random shooting); planning horizon must stay below $K$, constraining lookahead.
- **Single inner gradient step (GrBAL).** Adaptation capacity per step is limited; larger shifts rely on MPC re-planning to recover over subsequent steps.

## Open questions

- Can model **uncertainty** (probabilistic ensembles) be folded into the adaptive dynamics model to improve robustness and planning, as the authors suggest?
- How far can the timestep-as-task formulation be pushed toward *non*-locally-consistent regimes — detecting *when* the environment changed rather than assuming a window?
- Does the gradient-based vs recurrence-based trade-off (GrBAL's better generalization vs ReBAL's strength on longer informative sequences) have a principled selection rule per task?
- Can the same "meta-learn a prior for fast K-step-context adaptation" recipe transfer from locomotion dynamics to **manipulation of objects with rapidly changing, hard-to-identify dynamics** (e.g. a new rope) — adapting a *forward* model from a short probe rather than identifying explicit parameters?

## My take

This is the **meta-learned-dynamics-adaptation spine**: the canonical statement that you can sidestep the "globally accurate model" requirement of model-based RL by instead meta-training a dynamics prior that is *fast to re-fit* from the last few timesteps, then planning with the re-fitted model under MPC. The timestep-as-task reframing is the conceptual contribution — it converts MAML/recurrent meta-learning from an episodic, hand-designed-task setting into an *online, self-supervised* adaptation loop, which is exactly what makes it deployable on a real robot (the first such meta-RL deployment).

Its relevance to the rope/DLO arc of this wiki is structural, not topical. The whole "swing a new rope for a few seconds and adapt" pattern that [[iterative-residual-policy-goal-conditioned-dynamic]] and [[wiggle-go-system-identification-zero-shot]] pursue is, at its core, *the manipulation instance of GrBAL's question*: given a short window of recent interaction with an unknown system, how do you cheaply specialize a dynamics model to it? Nagabandi et al. (with [[chelsea-finn]], whose MAML this directly builds on) give the meta-RL formalization of that question for locomotion dynamics; IRP gives a non-meta, observation-conditioned residual-dynamics answer; Wiggle-and-Go gives an explicit-sysID answer. Reading the DLO papers against this one sharpens what is and isn't novel in each — IRP's delta-dynamics is a *different* mechanism (predict the trajectory delta from a single observation, no inner gradient/recurrent update of a forward model), but the *goal* (online local adaptation of dynamics from recent context) is shared, which is why the cross-edges below are `similar_method_to` rather than `builds_on`. For a DeformY follow-on, the cleanest open lever is exactly the fourth open question: meta-train a rope forward-dynamics prior (on a Cosserat-grade simulator) for fast K-step-context adaptation, then plan with MPC — GrBAL transplanted from a crippled cheetah to an unidentified rope.

## Related

- [[sim-to-real-and-rapid-adaptation]]
- [[model-based-planning-for-manipulation]]
**Foundations used**

- [[meta-learning]] — GrBAL builds on MAML; ReBAL builds on recurrent (black-box) meta-learners
- [[model-based-reinforcement-learning]] — the host paradigm; the method removes its global-model requirement
- [[model-predictive-control]] — MPPI (sim) / random shooting (real) plan with the adapted model each timestep
- [[system-identification]] — adapting a forward model online is closely related to online system identification (and contrasted with classical adaptive control / inverse-model adaptation)
- [[markov-decision-process]] — the control formalism
- [[probabilistic-ensemble]] — named as an orthogonal, complementary uncertainty mechanism the model could incorporate
- [[policy-gradient]] — model-free baselines (TRPO, MAML-RL) are policy-gradient methods, the comparison class for sample efficiency

**Methods introduced**

- [[grbal-rebal-online-meta-learned-dynamics-adaptation]] — the named, reusable procedure: meta-train a dynamics prior + update rule (GrBAL gradient step / ReBAL recurrent update), adapt from past M timesteps each step, plan with MPC

**Concepts introduced**

- [[online-meta-learned-dynamics-adaptation]] — the timestep-as-task idea: meta-learn a dynamics-model prior for fast within-episode adaptation from a short K-step context

**Cited works in this wiki**

- (none of this paper's bibliography is currently ingested as a wiki paper page)

**Cited by in this wiki**

- [[rma-rapid-motor-adaptation-legged-robots]] — RMA cites this paper and is the amortized context-encoder counterpart to GrBAL/ReBAL's online gradient/recurrent model re-fit (`similar_method_to`)
