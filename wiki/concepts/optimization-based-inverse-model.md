---
title: Optimization-Based Inverse Model (Norm-Optimal ILC)
aliases:
- norm-optimal ILC
- QP-based inverse model
- QP inverse model
- optimization-based ILC inverse
tags:
- iterative-learning-control
- ILC
- control
- optimization
- quadratic-program
- robot-learning
- inverse-model
maturity: active
key_papers:
- '[[learning-deformable-object-manipulation-using-task]]'
first_introduced: '2009'
date_updated: '2026-05-06'
related_concepts:
- '[[task-level-iterative-learning-control]]'
- '[[critical-point-objective]]'
---
## Definition

An **optimization-based inverse model** in ILC formulates the per-trial command update $\Delta \mathbf{u}(t)$ as the solution to a constrained quadratic program (QP). The QP minimizes a weighted task-error term plus a control-effort term, subject to linearized system-dynamics constraints and linearized actuator/state limits. This replaces the textbook closed-form pseudo-inverse $\Delta \mathbf{u} = \mathbf{M}^{\dagger} \tilde{\mathbf{x}}$ with a constrained optimization that explicitly handles input/state constraints (joint position, velocity, acceleration, torque) and lets the user specify *non-uniform* time-weighting of error.

Norm-Optimal ILC, introduced for robotics by Schoellig and colleagues (2009-2012) for aggressive quadrotor trajectory tracking, is the canonical optimization-based ILC formulation.

## Intuition

Computing $\mathbf{M}^{-1}$ analytically is fine when (a) $\mathbf{M}$ is well-conditioned, (b) the desired update lives inside the actuation limits, and (c) error weighting is uniform in time. None of these hold for dynamic deformable-object manipulation: the linearized rope dynamics are stiff and badly-conditioned, the inverse can demand commands beyond the arm's joint-velocity limits, and the cost is concentrated at a critical point. A QP fixes all three with one mechanism — minimize what you want, constrain what you cannot violate, and let the solver project onto the feasible set.

Choosing a QP also gives you free regularization: a control-effort penalty $\|\Delta \mathbf{u}\|^2_{\mathbf{R}}$ damps high-frequency modes that an unregularized inverse model would amplify (mechanical systems with energy loss are low-pass; their inverses are high-pass; ILC therefore amplifies high-frequency measurement noise without regularization).

## Formal notation

The optimization-based inverse model in [[learning-deformable-object-manipulation-using-task]] is:

$$\mathcal{M}^{-1}(\tilde{\mathbf{x}}_k(t)) = \min_{\Delta \mathbf{u}, \Delta \mathbf{x}} \;
\bigl\|\Delta \mathbf{x}(t_c) - \tilde{\mathbf{x}}_k(t_c)\bigr\|^2_{\mathbf{Q}}
+ \sum_{t \in [t_c, T]} \|\Delta \mathbf{u}(t)\|^2_{\mathbf{Q}_{\text{ft}}}
+ \sum_{t \in [0, T]} \|\Delta \mathbf{u}(t)\|^2_{\mathbf{R}}$$

subject to:

- $\Delta \mathbf{x}(t) = \mathbf{M} \, \Delta \mathbf{u}(t)$ (linearized system dynamics, $\mathbf{M} = \partial \mathcal{M}/\partial \mathbf{u}$ at $(\hat{\mathbf{x}}_k, \mathbf{u}_k)$)
- $\mathbf{q}_{\min} \le \mathbf{J}_p \Delta \mathbf{u}(t) + \mathcal{B}(\mathbf{u}(t)) \le \mathbf{q}_{\max}$ (joint position limits)
- $\dot{\mathbf{q}}_{\min} \le \mathbf{J}_v \Delta \mathbf{u}(t) + \dot{\mathcal{B}}(\mathbf{u}(t)) \le \dot{\mathbf{q}}_{\max}$ (joint velocity limits)
- $\ddot{\mathbf{q}}_{\min} \le \mathbf{J}_a \Delta \mathbf{u}(t) + \ddot{\mathcal{B}}(\mathbf{u}(t)) \le \ddot{\mathbf{q}}_{\max}$ (acceleration limits)
- $\tau_{\min} \le \mathbf{J}_\tau \Delta \mathbf{u}(t) + \mathcal{T}(\mathbf{u}(t)) \le \tau_{\max}$ (joint torque limits)

with $\mathbf{J}_p, \mathbf{J}_v, \mathbf{J}_a, \mathbf{J}_\tau$ Jacobians of joint position, velocity, acceleration, and torque about the current command.

## Variants

- **Norm-optimal ILC (Schoellig 2009, 2012)** — original formulation for quadrotor trajectory tracking; uniform time-weighting on tracking error.
- **Constrained ILC for industrial robots (Tan et al. 2007)** — earlier work applying norm-optimal ideas to industrial gantry tracking.
- **Critical-point QP (Suresh & Atkeson 2026)** — same formulation but task-tracking error is concentrated at one instant via [[critical-point-objective]].
- **Trust-region ILC** — bound $\|\Delta \mathbf{u}\|$ explicitly to stay inside the linearization's region of validity.
- **MPC-as-inverse-model** — receding-horizon QP rather than full-trajectory QP; relevant when state observations come online.

## When to use

- A linear or linearized system model is available.
- Hard input or state constraints (joint limits, actuation bounds) must not be violated.
- The error weighting is non-uniform in time (e.g. a critical-point objective).
- Solver speed is acceptable (typical Drake + Clarabel rope-flying-knot QP solves in milliseconds-to-seconds, fast relative to a real trial).

## Known limitations

- *Linearization range*: the QP solution is only optimal under the linearized dynamics; large $\Delta \mathbf{u}$ can step out of validity. Trust-region or step-size limits help.
- *Conditioning*: stiff systems produce ill-conditioned $\mathbf{M}$; numerical regularization (small $\mathbf{R}$) is often required.
- *No real-system line search*: the QP has no information about whether the chosen $\Delta \mathbf{u}$ actually decreases the real-system cost; cost can increase trial-to-trial after a bad linearization.
- *Solver dependency*: behavior depends on QP solver choice and warm-start strategy.

## Open problems

- Trust-region adaptation that learns the safe step size from past trials.
- Convexification techniques for non-quadratic task costs (e.g. topological success indicators).
- Online certificate of "the linearization gradient is in the right half-space" before committing the update.
- Replacing the QP with an SOCP or SDP when state constraints are non-polyhedral.

## Key papers

- [[learning-deformable-object-manipulation-using-task]] — uses a Drake-formulated, Clarabel-solved QP as the inverse model for Task-Level ILC of the flying knot, with a critical-point objective and full set of joint-dynamics constraints.

## My understanding

The QP-as-inverse-model is one of those engineering moves that looks bureaucratic but actually re-shapes what learning loops can do. Closed-form pseudo-inverses of unconstrained linear systems get you nowhere on a real arm: they ask for command updates that violate joint limits, demand infinite acceleration, or tear off into high-frequency noise on the first iteration. A QP swaps "find the analytic optimum, then post-process to feasibility" for "find the constrained optimum directly." For manipulation that is a structural improvement; it is also why the same Task-Level ILC algorithm can be paired with a critical-point objective without blowing up.

The interesting next move is *learning* the QP itself — the cost weights, possibly the linearization point, possibly the constraint set — across trials. That puts you halfway to MPC and is the natural way Task-Level ILC and modern model-based RL might converge.
