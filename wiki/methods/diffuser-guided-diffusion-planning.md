---
name: "Diffuser: Guided Diffusion Planning"
slug: "diffuser-guided-diffusion-planning"
type: inference
tags: [diffusion-model, planning, trajectory-optimization, model-based-reinforcement-learning, classifier-guidance, offline-rl, generative-planning, receding-horizon-control]
source_papers: ["[[planning-diffusion-flexible-behavior-synthesis]]"]
parent_methods: []
child_methods: []
realizes_concepts: ["[[planning-as-diffusion]]"]
code_repo: "https://github.com/jannerm/diffuser"
date_updated: 2026-06-16
---

## Problem setting

A data-driven agent must synthesize behavior — long-horizon, possibly multi-task, possibly goal-constrained — from a fixed dataset of feasible trajectories, *without* the model-exploitation pathology that arises when a strong trajectory optimizer is run against a learned single-step dynamics model. Diffuser targets settings that (a) provide offline trajectory data (demonstrations or mixed-quality logs) in a fixed environment, (b) require planning that is robust to long horizons and sparse rewards or to test-time goals unseen during training, and (c) can express the task either as a return/cost to maximize or as constraints (start, goal, contact) to satisfy. It assumes a known reward predictor or cost for guidance and tolerates iterative (non-real-time) generation. It learns **no separate dynamics model and no policy network** — a single trajectory diffusion model serves both roles.

## Mechanism

Diffuser is a **diffusion probabilistic model over whole trajectories of interleaved states and actions**, designed so that *sampling from it is nearly identical to planning with it*. Three design commitments make it a planner rather than a predictor:

- **Joint state+action trajectory model.** A plan is a 2-D array $\boldsymbol\tau=\big[\begin{smallmatrix}\mathbf s_0&\cdots&\mathbf s_T\\\mathbf a_0&\cdots&\mathbf a_T\end{smallmatrix}\big]$ (one column per planning step); states and actions are denoised *jointly*, with actions treated as extra state dimensions, because the quality of the induced controller matters as much as state prediction.
- **Non-autoregressive, temporally local denoiser.** All timesteps are predicted concurrently (decision-making is anti-causal: present actions condition on future desired states). The network is a U-Net of 1-D *temporal* convolutional residual blocks, so each denoising step sees only nearby past/future steps; composing many steps turns local consistency into **global coherence**. Being fully convolutional in the horizon dimension, plan length is set by the input-noise size, not the architecture — **variable-length plans at test time**.
- **Planning = guided sampling in a perturbed distribution.** Inference targets $\tilde p_\theta(\boldsymbol\tau)\propto p_\theta(\boldsymbol\tau)\,h(\boldsymbol\tau)$. Two reusable planning operators: **cost/classifier guidance** (shift the reverse-process mean by $\alpha\,\Sigma\,\nabla\mathcal J(\mu)$, the gradient of a return/cost — the control analogue of classifier guidance) and **inpainting** (fix observed entries — start state, goal — and fill in the rest). Multiple perturbations **compose** by summing gradients. Because dynamics live in $p_\theta$ and reward lives in $h$, one model is reused across tasks/rewards.

The distinctive, reusable design is the *coupling itself*: a generative trajectory prior keeps plans on the data manifold (defusing model exploitation) while lightweight guidance steers them — a different structural defense from the uncertainty-propagation route of [[pets-probabilistic-ensembles-trajectory-sampling]].

## Procedure

**Training (offline).**
1. Fit the trajectory diffusion model $\epsilon_\theta(\boldsymbol\tau^i,i)$ on all available trajectory data with the simplified DDPM objective $\mathcal L(\theta)=\mathbb E_{i,\epsilon,\boldsymbol\tau^0}\lVert\epsilon-\epsilon_\theta(\boldsymbol\tau^i,i)\rVert^2$; reverse covariances on the cosine schedule.
2. (For return-guided planning) Train a separate return predictor $\mathcal J_\phi$ on the same trajectories (it reuses the first half of the diffusion U-Net).

**Planning (Algorithm 1, "Guided Diffusion Planning"; receding horizon).** While not done:
1. Observe state $\mathbf s$; initialize the plan $\boldsymbol\tau^N\sim\mathcal N(\mathbf 0,\mathbf I)$.
2. For $i=N,\dots,1$: compute $\mu\gets\mu_\theta(\boldsymbol\tau^i)$; sample $\boldsymbol\tau^{i-1}\sim\mathcal N(\mu+\alpha\,\Sigma\,\nabla\mathcal J(\mu),\,\Sigma^i)$ (guidance); **overwrite the first state** $\boldsymbol\tau^{i-1}_{\mathbf s_0}\gets\mathbf s$ (inpaint the current state at every diffusion step). For pure goal-reaching, also overwrite the goal entry; for composed tasks, add each perturbation's gradient.
3. Execute the first action $\boldsymbol\tau^0_{\mathbf a_0}$; replan.

**Faster replanning (optional).** Warm-start the next plan by running a few *forward* diffusion steps from the previous plan, then denoising back — cutting the per-step denoising budget (≈100 → single digits) with only a modest performance drop.

Representative hyperparameters: guide scale $\alpha=0.1$ (most tasks; lower for short horizons); $N=20$ diffusion steps (locomotion), $N=100$ (block-stacking); planning horizon $T=100$ (locomotion), $128$ (block-stacking / large mazes); Adam, lr $4\!\times\!10^{-5}$, batch size 32.

## Assumptions

- A fixed dataset of **feasible trajectories** in the target environment is available for training (offline / demonstrations).
- A **known reward predictor $\mathcal J_\phi$ or cost** is available for guidance; goal/constraint tasks need a way to express constraints as fixed trajectory entries (inpainting).
- The guidance Gaussian approximation holds — i.e. $p(\mathcal O_{1:T}\mid\boldsymbol\tau^i)$ is **sufficiently smooth** over the diffusion trajectory.
- **Iterative (non-real-time) generation is acceptable**, or warm-starting suffices for the control rate.
- States (and actions) are an adequate trajectory representation; the environment is roughly stationary so one trajectory prior serves all tasks/rewards in it.

## Limitations

- **Slow generation** — each plan is an iterative denoising sweep; warm-starting mitigates but does not remove the cost; real-time closed-loop control is not demonstrated.
- **Only ties the best offline-RL methods on standard single-task benchmarks** (D4RL locomotion); its advantage is specific to long-horizon, multi-task, and test-time-flexible settings.
- **Needs a cost / return predictor** and a tuned **guide scale** $\alpha$ — the model+guidance balance is not automatic.
- **Guidance can push samples off-distribution** when offline data poorly covers the rewarded region.
- **Reusing the trajectory model inside a classical optimizer fails** — the authors' own MPPI-on-Diffuser variant "performed no better than random," so the gain is from coupled modeling+planning, not predictive accuracy.

## Tradeoff profile

- **Long-horizon / sparse-reward planning:** very strong — Maze2D single-task avg 119.5, multi-task avg 129.4 (vs. IQL 47.0 / 16.9; MPPI with true dynamics only 16.2 / 21.5), with zero retraining across single→multi-task.
- **Test-time flexibility / compositional goals:** strong — block stacking avg 54.4 (vs. CQL 8.1, BCQ 0.0), succeeding on conditional/rearrangement tasks where model-free offline RL scores 0.0.
- **Standard single-task offline RL:** comparable, not dominant — D4RL locomotion avg 77.5 (≈ CQL 77.6, IQL 77.0, TT 78.9).
- **Robustness to model exploitation:** high by construction — plans are sampled from the learned behavior manifold, so adversarial trajectories are low-probability.
- **Compute at control time:** high — iterative denoising per replan; mitigated by warm-starting.
- **Generality:** broad — one reward-agnostic trajectory prior reused across tasks and unseen rewards via guidance/inpainting; demonstrated on point mazes, MuJoCo locomotion, and Kuka block stacking.

## Evaluated by

- [[planning-diffusion-flexible-behavior-synthesis]] — Maze2D / Multi2D, D4RL locomotion, and Kuka block stacking: long-horizon planning (119.5 / 129.4 maze averages), test-time flexibility (54.4 block-stacking average), offline RL parity (77.5 locomotion average); the diagnostic that Diffuser-as-a-dynamics-model inside MPPI performs no better than random establishes that the effectiveness comes from coupled modeling and planning.
