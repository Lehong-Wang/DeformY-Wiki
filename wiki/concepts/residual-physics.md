---
title: "Residual Physics"
aliases: ["residual-physics", "residual on physics-based controller", "residual learning on analytical action prior", "residual policy on physics prior", "physics-residual hybrid controller"]
tags: [robotics, hybrid-controller, residual-learning, sim-to-real, action-space, manipulation, throwing]
maturity: active
key_papers: ["[[tossingbot-learning-throw-arbitrary-objects-residual]]"]
first_introduced: "2019"
date_updated: 2026-05-06
related_concepts: []
---

## Definition

**Residual Physics** is a hybrid controller pattern in which an analytical / physics-based controller produces a base estimate $\hat a$ of a control parameter (typically an action — release velocity, joint torque, end-effector twist, etc.) from the desired task state, and a learned function (usually a deep network) predicts an additive correction (the *residual*) $\delta$ on top of $\hat a$ such that the executed control is

$$a \;=\; \hat a \;+\; \delta(\text{state, context}).$$

The residual compensates for dynamics that the analytical model cannot capture (aerodynamics, contact, grasp-conditioned offsets, real-world noise) while the analytical part supplies cheap, structurally correct generalization to unseen task conditions (e.g. new target locations).

It is distinct from *residual state* models that learn a correction on the predicted **next state** of an analytical dynamics model (Ajay et al. 2018; Kloss et al. 2017). Residual Physics learns a correction on the **action**.

## Intuition

Two complementary sources of inductive bias:

- The analytical prior captures the *structurally correct* part of the dynamics (e.g. ballistic projectile motion is a good first approximation of aerial trajectories under gravity).
- The learned residual captures the *unmodeled remainder* (e.g. drag, grasp-induced offsets in release velocity, frictional contact).

Because the analytical part interpolates correctly across task parameters that *are* covered by the model (e.g. ballistics gives the right qualitative dependence on target distance), the policy can extrapolate to unseen targets without retraining; the residual only needs to be locally accurate around the analytical estimate. This is why Residual-Physics dramatically outperforms pure regression on novel targets even when both have seen similar object distributions.

## Formal notation

Let the task state be $s$ and the desired task condition (goal) be $g$. The analytical controller is a function $\pi_\phi: (s, g) \to \hat a$. The learned residual is $\delta_\theta(s, g, \mu(o))$, where $\mu(o)$ are perceptual features. Final control:

$$a_\theta(s, g, o) \;=\; \pi_\phi(s, g) \;+\; \delta_\theta(s, g, \mu(o)).$$

Training is end-to-end on a task-level loss $\mathcal L(a, y)$ where $y$ is task success / outcome. Often $\hat a$ is provided as input to the network so the residual can be conditioned on it.

In TossingBot specifically, the analytical part inverts the projectile equations $p = r + \hat v t + \tfrac12 a t^2$ for $\hat v$, then the network outputs a scalar $\delta$ such that $\|v\| = \|\hat v\| + \delta$.

## Variants

- **Action-space residual on closed-form controller** (TossingBot, Residual RL — Johannink et al., Silver et al. 2018): residual on the executed action.
- **Action-space residual on a learned base policy** (residual policy learning): the prior is itself a learned policy (e.g. a behavior-cloned baseline), and a small RL head learns the residual.
- **State-space residual** (Ajay et al., Kloss et al.): residual on the *next-state* prediction of an analytical or learned dynamics model — used for model-based control / planning.
- **Iterative residual** (IRP — Iterative Residual Policy): apply the residual update repeatedly across rollouts to converge on a goal-conditioned dynamic action.
- **Residual on an inverse kinematics / impedance controller**: residual on joint torques or end-effector wrenches relative to a model-based reference.

## Comparison

- **vs. pure data-driven regression of the action.** Same learning capacity, but no analytical prior — does not generalize to unseen task conditions outside the training distribution; needs more data; less sample-efficient.
- **vs. pure analytical / model-based control.** Generalizes well to new task conditions but cannot compensate for the un-modelled physical effects that dominate accuracy in practice (aerodynamics, contact, grasp).
- **vs. residual *state* models.** State residuals improve simulator fidelity for planning; action residuals modify the executed control directly. Action residuals provide a wider range of corrections at the cost of needing closed-loop / online experience.
- **vs. domain randomization / system identification.** Both close the sim-to-real gap, but DR/SysID stay model-based; Residual Physics injects expressive learning capacity into the controller itself.

## When to use

- A reasonably good analytical / model-based controller exists for the task (even if its assumptions are violated in practice).
- The unmodeled effects (drag, grasp, contact) materially affect task success.
- Generalization across task conditions (target locations, object types, scenes) is required and learning from scratch would be data-prohibitive.
- A low-dimensional, *task-relevant* action representation is available (or can be constructed) — Residual Physics is most effective when the residual lives in a small subspace.

Avoid when the analytical prior is so wrong that the residual must dominate (then pure regression is comparable and simpler), or when actions cannot be decomposed additively (e.g. discrete / mode-switching control).

## Known limitations

- **Quality bounded by the prior's task-relevant correctness.** If the analytical model is wrong about the task-relevant structure (not just the magnitude), the residual cannot recover.
- **Interaction with the prior is opaque.** Hard to inspect what the residual has learned; failure modes can correlate with grasp / state regions where the residual is large.
- **Single-task demonstrations.** Most published Residual Physics results are within a narrow task family (throwing, planar pushing, peg-in-hole). General multi-task transfer is open.
- **Cannot compensate for missing modes.** If the analytical controller cannot represent a control mode the task requires (e.g. switching between under- and over-arm throws), the residual cannot synthesize it.

## Open problems

- How to **automatically discover** an appropriate analytical prior given only a task description and a simulator.
- **Targeted-pose residuals**: extending residual physics from scalar / velocity residuals to full 6-DoF release pose, with consistency constraints.
- **Residual on residual / hierarchical**: stacking residuals on top of *learned* dynamics models rather than only closed-form ones, to recover differentiability and gradient-based planning.
- Quantitative theory of when residual physics provably **dominates** pure regression in sample complexity, especially under noisy priors.
- Application to **deformable-object dynamics** (DLO, cloth) where the analytical prior is itself partial and the residual must compensate over distributed state.

## Key papers

- [[tossingbot-learning-throw-arbitrary-objects-residual]] — coins the term "Residual Physics", demonstrates 84.7% real-world throw accuracy on arbitrary objects with action-space residuals on a ballistic prior.

## My understanding

Residual Physics is the cleanest hybrid-control template currently available for task-driven robot learning, and it is the methodological substrate I expect DeformY-style dynamic DLO control to ride on. The key design decisions are (a) keep the analytical prior simple enough to be reliable across task conditions, (b) feed the prior's output into the network as a feature so the residual is conditioned on the regime it is correcting, and (c) use task-level supervision (did the throw / swing / cast succeed?) so the residual is trained against the same metric the user cares about. For DeformY's tip-targeting problem on a flexible rod, the analogous prior could be a Cosserat-rod-derived release condition for a desired tip trajectory, with the residual covering grip-induced offsets and contact perturbations the rod model omits. The IRP (iterative residual) variant likely matters more than the one-shot variant in the DLO regime because the residual itself is high-dimensional.
