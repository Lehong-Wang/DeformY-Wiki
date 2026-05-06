---
title: Critical-Point Objective
aliases:
- critical point objective
- single-point trajectory cost
- event-focused ILC objective
tags:
- iterative-learning-control
- ILC
- control
- deformable-object-manipulation
- trajectory-cost
- behavior-cloning
- robot-learning
maturity: emerging
key_papers:
- '[[learning-deformable-object-manipulation-using-task]]'
first_introduced: '2026'
date_updated: '2026-05-06'
related_concepts:
- '[[task-level-iterative-learning-control]]'
- '[[optimization-based-inverse-model]]'
---
## Definition

The **critical-point objective** is a trajectory-cost design choice in which task-tracking error is weighted only at a manually- or automatically-identified key moment $t_c$ in the task, rather than weighted equally (or by a smooth weighting kernel) along the entire trajectory. Concretely, given a task-state trajectory $\mathbf{x}(t)$, a reference $\mathbf{x}^{\text{ref}}(t)$, and a critical instant $t_c \in [0, T]$:

$$J_{\text{crit}}(\mathbf{x}(\cdot), \mathbf{x}^{\text{ref}}(\cdot)) \;=\; \bigl\|\mathbf{x}(t_c) - \mathbf{x}^{\text{ref}}(t_c)\bigr\|^2_{\mathbf{Q}}.$$

Errors at $t \ne t_c$ are not in the cost. The objective can be combined with an action-effort term and with secondary tracking on $t > t_c$ (e.g. follow-through) at a much smaller weight.

## Intuition

For tasks where success or failure is decided at a single conspicuous instant — a contact event, a release moment, a collision — most of the trajectory does not need to match the demonstration; only the rope shape (or end-effector pose, or object configuration) at the decisive instant does. Equal-weighting cost wastes the inverse-model's budget reducing error on irrelevant earlier samples, and *increases* error at the decisive moment as a side effect, because the linearized inverse model sums over time and trades off across samples.

The critical point also acts as a *task abstraction*: it strips out the complicated post-event physics (rope-rope collisions, friction, follow-through) by simply not asking the model to predict them. For manipulation of deformable objects this is huge — accurate post-contact rope models are exactly what the field does not have.

## Formal notation

Given an inverse-model QP that solves for a command update $\Delta \mathbf{u}(t)$:

$$\Delta \mathbf{u}^* \;=\; \arg\min_{\Delta \mathbf{u}, \Delta \mathbf{x}} \;
\underbrace{\bigl\|\Delta \mathbf{x}(t_c) - \tilde{\mathbf{x}}_k(t_c)\bigr\|^2_{\mathbf{Q}}}_{\text{critical-point term}}
\;+\;
\sum_{t \in [t_c, T]} \|\Delta \mathbf{u}(t)\|^2_{\mathbf{Q}_{\text{ft}}}
\;+\;
\sum_{t \in [0, T]} \|\Delta \mathbf{u}(t)\|^2_{\mathbf{R}}$$

subject to linearized dynamics $\Delta \mathbf{x}(t) = \mathbf{M} \, \Delta \mathbf{u}(t)$ and linearized actuation/state constraints. Only the critical-point error appears in the *task-tracking* term; $\mathbf{Q}_{\text{ft}}$ is a small follow-through fingertip-tracking penalty after $t_c$, and $\mathbf{R}$ is control effort.

## Variants

- **Single critical point** — used in flying-knot work; $t_c$ is the rope self-collision instant.
- **Multiple critical points** — natural extension for tasks with several decisive sub-events; not yet evaluated.
- **Critical-region objective** — soft window around $t_c$ rather than a hard delta-function in time.
- **Auto-discovered critical points** — pick $t_c$ from contact-state changes, dynamical extrema (velocity, acceleration, jerk peaks), branch points in the trajectory, or human instruction. Open problem.
- **Behavior-cloning analogue** — weighted sub-goal supervision in BC (e.g. instructor-guided sub-goals) is structurally similar to a critical-point objective.

## Comparison

Compared to standard equally-weighted trajectory tracking (the default in trajectory-tracking ILC and most behavior cloning):

- Equal-weighted: low error early, high error late; can fail at the decisive instant even when overall MSE is small.
- Critical-point: high error early-and-late, low error at $t_c$; succeeds at the decisive instant by design.

The flying-knot evaluation is a clean A/B: same hardware, same model, same demo, same algorithm modulo the cost term — equal-weighted fails, critical-point succeeds.

## When to use

- The task is success/failure-defined at one (or a few) conspicuous instants.
- Post-event dynamics are hard to model and not load-bearing for success.
- A demonstration or measurement provides a reference task state at the critical instant.
- The cost over $t \ne t_c$ is genuinely irrelevant to task success.

## Known limitations

- *Critical point must be specified.* If $t_c$ is wrong (the rope contacts earlier than the time-indexed critical point assumes), the inverse model reads stale state and the update direction is wrong.
- *Past-$t_c$ behavior is uncontrolled.* Successful loops can untie themselves in follow-through if the cost is silent there.
- *Marker-tracking failures* near contact events make the critical point hard to *measure* exactly when it is most decisive.
- *Not appropriate for tasks where the entire trajectory matters* (e.g. precise contour-following).

## Open problems

- Autonomous selection of critical points from demonstrations or instructional materials.
- Multi-critical-point objectives for multi-sub-event tasks (e.g. multi-step knots, sequential assembly).
- Critical-region (soft window) versus critical-point (hard delta) — which converges faster, which generalizes farther?
- Connection to event-triggered control and hybrid systems theory.
- Lifting the same idea into Behavior Cloning for sample-efficient demonstration learning.

## Key papers

- [[learning-deformable-object-manipulation-using-task]] — introduces the critical-point objective for Task-Level ILC and demonstrates it is *required* (not just helpful) for learning the flying knot.

## My understanding

The critical-point objective looks small but is doing a lot of conceptual work. It is simultaneously (1) a way to throw away unmodeled dynamics that you cannot afford to simulate, (2) a way to focus update budget on the moment that decides success, and (3) a *task abstraction* — declaring that the task essentially is "achieve this rope shape at this instant," everything else is means. That last reading is the most generative: it suggests that one of the bottlenecks in dynamic deformable-object manipulation is that we have not articulated what the task even *is*, and pointing at one frame from a demo is a surprisingly powerful way to do that. Whether the trick generalizes beyond rope to cloth and articulated multi-event tasks is the obvious open question.
