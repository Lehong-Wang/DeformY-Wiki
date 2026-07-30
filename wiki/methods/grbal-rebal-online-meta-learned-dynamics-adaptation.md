---
name: "GrBAL / ReBAL: Online Meta-Learned Dynamics Adaptation"
slug: "grbal-rebal-online-meta-learned-dynamics-adaptation"
type: training
tags: [meta-reinforcement-learning, model-based-reinforcement-learning, online-adaptation, MAML, recurrent-model, MPC, dynamics-model, sim-to-real]
source_papers: ["[[learning-adapt-dynamic-real-world-environments]]"]
parent_methods: []
child_methods: []
realizes_concepts: ["[[online-meta-learned-dynamics-adaptation]]"]
code_repo: "https://sites.google.com/berkeley.edu/metaadaptivecontrol"
date_updated: 2026-06-16
---

## Problem setting

Model-based RL agents must act in environments whose dynamics change at run time (component failure, terrain, payload, miscalibration), where (a) a single globally accurate dynamics model is impractical or impossible, and (b) re-learning after each change is too slow for real-time control. The method targets continuous-control agents that can collect a short window of recent transitions each timestep and plan with a sampling-based MPC controller — including real robots, where sample efficiency is paramount.

## Mechanism

Meta-train two sets of parameters for a forward dynamics model $\hat p_\theta(\mathbf{s}'\mid\mathbf{s},\mathbf{a})$:

- a **model prior** $\theta$, and
- an **update rule** $u_\psi$ that maps a recent trajectory segment to adapted parameters $\theta' = u_\psi(\tau(t{-}M,t{-}1), \theta)$.

They are trained jointly against a meta-objective that adapts on the past $M$ timesteps and is scored on the next $K$:
$$\min_{\theta,\psi}\ \mathbb{E}_{\tau_\mathcal{E}(t-M,t+K)\sim\mathcal{D}}\big[\mathcal{L}(\tau_\mathcal{E}(t,t+K),\,\theta'_\mathcal{E})\big],\qquad \mathcal{L}=-\tfrac1K\!\!\sum_{k=t}^{t+K}\!\log\hat p_{\theta'_\mathcal{E}}(\mathbf{s}_{k+1}\mid\mathbf{s}_k,\mathbf{a}_k).$$

Two instantiations of $u_\psi$:

- **GrBAL (gradient-based).** A MAML inner gradient step on the recent-data likelihood: $\theta'=\theta+\psi\,\nabla_\theta\frac1M\sum_{m=t-M}^{t-1}\log\hat p_\theta(\mathbf{s}_{m+1}\mid\mathbf{s}_m,\mathbf{a}_m)$, with $\psi$ a learnable/fixed inner learning rate; the outer loop backprops the meta-loss through this step. A single inner step is used.
- **ReBAL (recurrence-based).** $u_\psi$ is a recurrent cell's hidden-state update; $\psi$ are the recurrent weights, $\theta$ the rest of the network and the hidden state. The update rule is itself learned through the cell's gating.

The defining design choice is the **timestep-as-task** formulation: tasks are length-$(M{+}K)$ trajectory *segments* drawn automatically from experience, not hand-designed episodes — enabling within-episode, online adaptation.

## Procedure

1. **Meta-train (Alg. "train time").** Initialize $\theta$. Periodically (every $n_S$ iters) sample an environment $\mathcal{E}\sim\rho(\mathcal{E})$ and collect a rollout *using the online-adaptation procedure itself* (on-policy data); add to dataset $\mathcal{D}$. Each iteration, sample $N$ segment pairs $\big(\tau(t{-}M,t{-}1),\tau(t,t{+}K)\big)$, compute adapted $\theta'$ via $u_\psi$, accumulate the meta-loss $\mathcal{L}_j$, and take outer gradient steps on $\theta$ (rate $\beta$) and $\psi$ (rate $\eta$).
2. **Deploy / adapt online (Alg. "test time").** At each timestep $t$: adapt $\theta_*' = u_{\psi_*}(\mathcal{D}(t{-}M,t{-}1),\theta_*)$; pass $\hat p_{\theta_*'}$, the reward, and planning horizon $H$ to the controller; execute the first action; append the transition to $\mathcal{D}$; **reset parameters to $\theta_*$**; repeat.
3. **Planner.** Sampling-based MPC — **MPPI** (model-predictive path integral) in simulation, **random shooting** on the real robot. Keep planning horizon $H < K$ (the adapted model is only locally valid). Re-planning every step compensates for residual model error.

Representative hyperparameters: neural dynamics model 3×512 ReLU, Gaussian mean / fixed variance (MLE = MSE); $K{=}M$ in the 16–32 range; a single GrBAL inner step; meta-training equivalent to ~1.5–3 hours of real-world experience.

## Assumptions

- **Locally consistent environment:** dynamics are approximately constant over each length-$(M{+}K)$ window (justified because adaptation is sub-second).
- **Shared structure across environments:** common state/action spaces; $\rho(\mathcal{E})$ varies only the dynamics.
- A short window of recent transitions is **informative** about the current dynamics regime.
- A reasonable reward/cost function and a sampling-based MPC controller are available at deployment.

## Limitations

- Adaptation capacity per step is bounded (e.g. one inner gradient step); large abrupt shifts are absorbed gradually via MPC re-planning.
- Inherits model-based RL's asymptotic sub-optimality vs the best model-free methods (seen on Ant-crippled).
- Fixed-variance model with no uncertainty; ensembles/uncertainty are complementary but not included.
- Performance depends on planner choice and on the window/horizon hyperparameters $M,K$ (robust but not free).
- The locally-consistent assumption breaks for changes occurring *within* a single adaptation window.

## Tradeoff profile

- **Sample efficiency:** very high — ~1000× less data than model-free / model-free-meta-RL baselines to reach comparable returns, by virtue of being model-based.
- **Adaptation speed:** fast (sub-second; a handful of timesteps), and faster than test-time dynamic evaluation of a non-adaptive model because the prior is trained *for* adaptability.
- **Robustness vs static models:** can exceed an MB oracle (trained with unlimited data on the test environment) in stochastic/disturbance settings, where no static model is robust.
- **GrBAL vs ReBAL:** GrBAL generalizes and fast-adapts better overall and is the variant deployed on the real robot; ReBAL is stronger when longer input sequences better reveal the current setting.
- **Compute:** adds an inner update + a sampling-based MPC solve per timestep.

## Evaluated by

- [[learning-adapt-dynamic-real-world-environments]] — MuJoCo half-cheetah (disabled joint / slope / pier) and Ant (crippled leg), plus a real 6-legged VelociRoACH millirobot adapting online to a missing leg, novel terrains/slopes, pose miscalibration, and pulled payloads; the first meta-RL method deployed on a real robot.
