---
title: Task-Level Iterative Learning Control
aliases:
- Task-Level ILC
- task-level ILC for manipulation
tags:
- iterative-learning-control
- ILC
- control
- deformable-object-manipulation
- robot-learning
- real-world-learning
maturity: emerging
key_papers:
- '[[learning-deformable-object-manipulation-using-task]]'
first_introduced: '2026'
date_updated: '2026-05-06'
related_concepts:
- '[[critical-point-objective]]'
- '[[optimization-based-inverse-model]]'
parent_topic: "[[dynamic-dlo-tip-targeting]]"
---
## Definition

**Task-Level Iterative Learning Control (Task-Level ILC)** is a variant of model-based Iterative Learning Control in which the iterative command-update loop minimizes error in the *manipulated object's* state — typically a small set of high-level task variables — rather than the robot's own joint or end-effector trajectory. The robot executes a feedforward command $\mathbf{u}_k(t)$ on hardware, the trial produces a measured task state trajectory $\mathbf{x}_k(t)$, and an inverse model maps the task-space error $\tilde{\mathbf{x}}_k(t) = \mathbf{x}_k - \mathbf{x}^{\text{ref}}$ into a command update $\Delta \mathbf{u}_k(t)$ via $\mathcal{M}^{-1}$. The next-trial command is $\mathbf{u}_{k+1}(t) = \mathbf{u}_k(t) - \Delta \mathbf{u}_k(t)$. Convergence does not require an accurate forward model — only a forward model whose linearization gives the *correct gradient direction* for $\partial \mathcal{M}/\partial \mathbf{u}$ in the relevant region of command space.

In manipulation, "task-level" specifically distinguishes the method from trajectory-tracking ILC, which corrects only the robot's own kinematic trajectory. Task-Level ILC explicitly closes the loop on the manipulated object — even when that object has many more (and largely unactuated) degrees of freedom than the robot itself.

## Intuition

ILC is "a little bit of feedback per trial" for repeatable open-loop tasks. Classical ILC corrects the robot's own trajectory so it tracks a reference more accurately each iteration. That is a sensible objective when the robot is the system you care about (e.g. a CNC arm tracking a contour). It becomes the *wrong* objective when the goal is to manipulate a deformable, underactuated object: the robot's trajectory is not the task; the rope's shape at a key instant is. Task-Level ILC reformulates the inverse-model step so each trial's update reduces error on the task quantity, even if doing so requires the robot's trajectory to drift further from the demonstration.

The other half of the intuition is that linearization error in the rope dynamics is acceptable as long as the gradient direction is right. Even very loose deformable models (5-parameter point-mass-with-bending chains; see [[mass-spring-system]]) carry enough information to steer command updates in the correct direction.

## Formal notation

Let $\mathcal{M}: \mathbf{u}(t) \mapsto \hat{\mathbf{x}}(t)$ be a forward system model from feedforward command to predicted task state. At iteration $k$:

1. Execute $\mathbf{u}_k(t)$ on hardware, measure $\mathbf{x}_k(t)$.
2. Compute task-level error $\tilde{\mathbf{x}}_k(t) = \mathbf{x}_k(t) - \mathbf{x}^{\text{ref}}(t)$.
3. Compute command update $\Delta \mathbf{u}_k(t) = \mathcal{M}^{-1}(\tilde{\mathbf{x}}_k(t))$, where $\mathcal{M}^{-1}$ is some pseudo-inverse — e.g. $\mathbf{M}^{\dagger}$ over the linearized Jacobian $\mathbf{M} = \left.\partial \mathcal{M} / \partial \mathbf{u}\right|_{(\hat{\mathbf{x}}_k, \mathbf{u}_k)}$, or the solution of a Quadratic Program (see [[optimization-based-inverse-model]]).
4. Update $\mathbf{u}_{k+1}(t) = \mathbf{u}_k(t) - \Delta \mathbf{u}_k(t)$.

The defining feature is that $\tilde{\mathbf{x}}$ is on *task-level* state — manipulated-object positions, contact states, or similar — not on robot-internal state.

## Variants

- **Trajectory-tracking ILC** (the older, classical case): $\tilde{\mathbf{x}}$ is the robot's joint or end-effector tracking error.
- **Task-Level ILC with critical-point objective** (Suresh & Atkeson 2026): the cost on $\tilde{\mathbf{x}}$ is concentrated at one decisive instant $t_c$; see [[critical-point-objective]]. This is the formulation that achieves <10-trial real-world learning of the flying knot.
- **Norm-Optimal Task-Level ILC**: $\Delta \mathbf{u}$ is computed via a QP that incorporates input/state constraints (joint limits, torque limits) — see [[optimization-based-inverse-model]].
- **Event-based ILC**: events (contact transitions) define when error is measured rather than continuous time.

## When to use

- The task is *repeatable* (deterministic apart from small noise) and feedforward.
- A loose forward model is available — accurate enough to give correct gradient direction, not necessarily accurate enough to predict states.
- Real-system trials are cheap relative to the cost of building a sim-to-real pipeline.
- The task quantity is observable on hardware (e.g. via motion capture).
- Robust closed-loop feedback is unnecessary or impractical (no large stochastic disturbances).

## Known limitations

- *Feedforward only*: Task-Level ILC learns $\mathbf{u}(t)$, not $\mathbf{u}(\mathbf{x})$. Unstable systems and large stochastic disturbances are out of scope without an external stabilizer.
- *Repeatability assumption*: errors must be reproducible across trials, otherwise update directions are noise-dominated.
- *Inverse model can amplify high-frequency error*: mechanical low-pass systems have high-pass inverses, and noisy task measurement gets amplified into the command.
- *Local minima and divergence*: with no real-system line search, command updates can degrade performance after a poor step.

## Open problems

- Automated discovery of the right *task variable* to put in $\tilde{\mathbf{x}}$ — e.g. when there are several plausible critical points, which one yields convergent learning?
- Quantitative theory for "loose-enough" forward models — when is the linearization gradient *guaranteed* to point in the right half-space?
- Hybrid Task-Level ILC + closed-loop policy: use ILC for the open-loop "skill" and a thin feedback policy for residual rejection.
- Multi-task Task-Level ILC: amortize the inverse model across related tasks (different rope types, different demonstrations).

## Key papers

- [[learning-deformable-object-manipulation-using-task]] — first application of Task-Level ILC to dynamic deformable-object manipulation; introduces the critical-point objective formulation.

## My understanding

Task-Level ILC is most usefully understood as the bridge between two communities. From the *control* side it is just ILC with the cost moved off the robot's body; from the *robot-learning* side it is the simplest possible "model-based, real-world, sample-efficient" learner — one demo, one QP, one loop. Its scope is narrower than RL or BC (feedforward only, repeatable tasks only), but inside that scope its sample efficiency is hard to beat. The 100% success in <10 trials on flying-knot is the existence proof.

The deeper bet is that *for a class of dynamic deformable-object tasks, the right move is not to build a better simulator but to build a worse one and put ILC around the real robot.* This is in direct conceptual tension with simulator-heavy lines of work like [[deformx-versatile-co-simulation-framework-deformable]] and [[rapid-adaptation-particle-dynamics-generalized-deformable]]. It will be interesting to see whether Task-Level ILC's empirical wins generalize to cloth, multi-step manipulation, and tasks where the critical point is less obvious.
