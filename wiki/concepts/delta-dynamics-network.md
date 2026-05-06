---
title: "Delta Dynamics Network"
aliases: ["delta dynamics model", "residual dynamics network", "delta-action trajectory predictor", "trajectory residual predictor"]
tags: [DLO, dynamic-manipulation, residual-learning, learned-dynamics, sim-to-real, robot-learning]
maturity: emerging
key_papers: ["[[iterative-residual-policy-goal-conditioned-dynamic]]"]
first_introduced: "2022"
date_updated: 2026-05-06
related_concepts: ["[[iterative-residual-policy]]"]
---

## Definition

A **delta dynamics network** is a learned model $f_\theta: (T_i, \delta a_i) \mapsto \hat{T}_i^{(\delta a_i)}$ that takes (i) an *observed* trajectory $T_i$ produced by some action $a_i$ and (ii) a small *delta action* $\delta a_i$, and predicts the trajectory that would result from action $a_i + \delta a_i$. The network does not model absolute dynamics — it only models how trajectories *change* under small action perturbations, conditioned on the most recent observation. This conditioning lets the network implicitly identify the system from the observed trajectory rather than relying on explicit physical parameters.

## Intuition

Modeling full forward dynamics of a deformable object requires capturing every relevant physical property (stiffness, density, aerodynamics, friction). Modeling *delta* dynamics requires only the local Jacobian-like sensitivity: "if I push action $a$ slightly in direction $\delta a$, how does the trajectory shift?". This is a much smaller, smoother, and more transferable function. The observed trajectory $T_i$ does the heavy lifting of telling the network what kind of object it is currently dealing with, so the network can focus on the residual.

## Formal notation

Given a parameterized action $a \in \mathbb{R}^{N_a}$ and a trajectory representation $T \in \mathbb{R}^{H \times W \times C}$ (e.g., a rasterized image of trajectory keypoints), the delta dynamics network is

$$
\hat{T}_i^{j} = f_\theta\bigl(T_i,\, \delta a_i^{j}\bigr)
$$

with the loss (binary cross-entropy when trajectories are binary occupancy maps):

$$
\mathcal{L}(\theta) = \mathbb{E}_{(T_i, \delta a_i, T_i^*)}\bigl[\mathrm{BCE}(f_\theta(T_i, \delta a_i),\, T_i^*)\bigr]
$$

where $T_i^*$ is the ground-truth trajectory under $a_i + \delta a_i$. Training data is synthesized by running pairs of actions $(a_i, a_i + \delta a_i)$ in a simulator over a (small) parameter grid of object instances.

## Variants

- **Image-trajectory + broadcast-action input** (the original IRP). Trajectory rasterized to a 2D image; delta action broadcast to $N_a$ image channels and concatenated; CNN backbone (DeepLabV3+).
- **Sequence-trajectory + concatenated-action input.** Trajectory as a sequence of keypoints; transformer / 1-D CNN backbone.
- **Multi-step delta dynamics.** Predict trajectories several iterations ahead, enabling beam-search / lookahead in the IRP loop.
- **Probabilistic delta dynamics.** Output a distribution over trajectories (variance, ensemble, mixture density) to capture multi-modality.
- **Differentiable delta dynamics.** Make $f_\theta$ smoothly differentiable in $\delta a$ to support gradient-based action optimization in addition to (or instead of) sampling.

## Comparison

- vs. **Forward dynamics model.** A forward model maps $(s_t, a_t) \to s_{t+1}$ over the entire dynamical horizon and tries to capture the full system. Delta dynamics maps a *trajectory-pair shift* and exploits the existing observation as conditioning.
- vs. **Inverse dynamics / direct action regression (DeltaReg).** Direct regression of an optimal action conflates multiple valid solutions (mode collapse). Delta dynamics evaluates many candidate deltas and is robust to multi-modality.
- vs. **Linearized plant model (iterLinear).** A linear approximation of the plant fitted online. Delta dynamics is non-linear (deep CNN) and can capture mode-switching behavior the linear model cannot.
- vs. **Differentiable simulator (DiffPhys).** A differentiable physics engine gives gradients through the full system but is expensive and requires accurate physics. Delta dynamics gets *empirically* the same gradient information from data, with no physics commitment.

## When to use

- Goal-conditioned manipulation tasks where a parameterized action primitive captures the task.
- Tasks with **repeatable** dynamics where iterative refinement is feasible.
- Settings where the **observed trajectory carries rich physical signal** (object response varies systematically with action).
- Sim-to-real settings where you have approximate but not exact simulators.

Skip when (a) the action space is high-dimensional and continuous (e.g., per-timestep torques) — the delta-dynamics formulation does not naturally extend to closed-loop control, or (b) the task is one-shot — there's no iteration.

## Known limitations

- Predicts only small perturbations well — too-large $\delta a$ takes you out-of-distribution and predictions degrade.
- Depends on a specific action parameterization; new action dimensions require re-training.
- Open-loop within a single action — does not produce within-action feedback.
- Greedy selection of best delta loses information; multi-modal cases may benefit from explicit distribution modeling.
- Trajectory representation matters: the IRP image-rasterization works for 2D motion but does not directly extend to 3D.

## Open problems

- Generalization to **3D action spaces** and 3D trajectory representations (point clouds, neural fields).
- Differentiable formulations that combine sampling with gradient-based optimization.
- Multi-step / beam-search prediction for long-horizon delta refinement.
- Combining delta dynamics with **Cosserat-rod-grade simulators** (e.g., the DeformX backend) to shrink the residual the network has to learn.
- Hybrid closed-loop + iterative-residual policies — within-action neural control plus across-action delta refinement.

## Key papers

- [[iterative-residual-policy-goal-conditioned-dynamic]] — introduces the architecture (DeepLabV3+ on stacked trajectory + broadcast-action images) and demonstrates that delta-dynamics prediction generalizes far better than full system identification on dynamic deformable manipulation.

## My understanding

The delta-dynamics network is the **engine** that makes [[iterative-residual-policy]] work. Its conceptual virtue is shifting the modeling burden from "predict the world from action" (a hard generative problem) to "predict trajectory shifts from action perturbations" (a much smaller and smoother regression problem, conditioned on the most informative possible signal — the actual observed trajectory). For DeformY, the open question is whether this trick scales: does a delta-dynamics network conditioned on a 3D trajectory representation, trained on a faithful Cosserat simulator, unlock real-time 3D dynamic deformable manipulation, or does the action-space dimensionality break the formulation?
