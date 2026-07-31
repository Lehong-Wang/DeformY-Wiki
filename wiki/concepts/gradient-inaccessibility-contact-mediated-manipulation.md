---
title: "Gradient Inaccessibility in Contact-Mediated Manipulation"
aliases: ["gradient inaccessibility", "zero-gradient-before-contact", "vanishing gradients in differentiable manipulation", "first-order failure in tool use", "non-informative simulator gradients"]
tags: [differentiable-physics, differentiable-simulation, trajectory-optimization, contact-rich-manipulation, tool-use, zeroth-order-optimization, CMA-ES, sparse-reward, DLO]
maturity: emerging
definition: "In differentiable simulators, the analytic gradient of a task reward with respect to actions is identically zero whenever the manipulated object has not yet made contact with the reward-carrying object, so first-order optimizers are structurally blind on tool-use and indirect-manipulation tasks no matter how exact the simulator's derivatives are."
key_papers: ["[[dlo-lab-benchmarking-deformable-linear-object]]", "[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]"]
first_introduced: "2026"
date_updated: 2026-07-30
related_concepts: ["[[differentiable-deformable-benchmark]]"]
parent_topic: "dynamic-deformable-object-simulation"
---

## Definition

A differentiable simulator promises `∂r/∂a` — the derivative of task reward with respect to the
control sequence. **Gradient inaccessibility** is the observation that this promise is empty on a
large and important class of tasks: whenever the reward is a function of an object the actuated
body has not yet touched, `∂r/∂a` is *exactly* zero, not merely small. The simulator is fully
differentiable and the gradient it returns is fully correct — and useless. First-order
optimization cannot start.

This is distinct from the more familiar complaint that contact gradients are *noisy* or
*biased*. Both failure modes appear in the same tasks and are often conflated, so it is worth
separating them:

| Failure mode | Where it lives | Symptom | Remedy |
|---|---|---|---|
| **Inaccessibility** (this concept) | *before* first contact | gradient is identically 0; optimizer never leaves its initialization | warm-start into contact; zeroth-order search; reward shaping that reaches back before contact |
| **Ruggedness / bias** | *during* intermittent contact | gradient exists but is discontinuous, high-variance, or points into a local optimum | smoothing, randomized smoothing, hybrid 0th/1st-order estimators |

## Intuition

Consider a rope used to drag a cube. The reward is the cube's displacement. Vary the gripper
trajectory by ε: if the rope never reaches the cube in either rollout, the cube's final position
is bit-identical, so the finite difference is zero and the analytic derivative agrees. There is no
signal *anywhere* in a neighbourhood of the initialization — the reward landscape is locally flat,
and flatness is not something a better gradient estimator can fix.

Sampling-based optimizers are immune for a structural reason: they only need the reward to differ
across *some* pair of samples in the population, not across an infinitesimal perturbation. With a
population of a few hundred parallel rollouts, some sample makes contact, and the search
distribution moves toward it.

The corollary that matters for method design: **the value of differentiability is conditional on
the task's contact structure, not on the simulator's quality.** A simulator can be perfectly
differentiable and still be best driven by CMA-ES.

## Formal notation

Let `s_T = f(a_{1:T}; s_0)` be the simulator rollout and `r(s_T)` the reward. Suppose the reward
depends on the state only through a subset of bodies `O` (the target objects), and let
`τ_c(a_{1:T})` be the first timestep at which the actuated system reaches `O`. If `τ_c = ∞` for
all `a` in a neighbourhood `N(a⁰)`, then

$$ \frac{\partial r}{\partial a}\bigg|_{a \in N(a^0)} \equiv 0 . $$

The set of trajectories that achieve contact has positive measure but is not reachable by
gradient flow from `a⁰`; it is reachable by a sampling distribution whose support covers it. Note
that the condition is on `N(a⁰)`, so the concept is really about *initialization coverage*: an
initialization that already grazes `O` restores a usable gradient.

## Variants

- **Pre-contact flatness (canonical).** Tool use and indirect manipulation: rope-drags-cube,
  slingshot-launches-ball, rope-lifts-ring. [[dlo-lab-benchmarking-deformable-linear-object]]'s
  *Gathering*, *Lifting* and *Slingshot*.
- **Anti-shortcut reward penalties turn contact into a hard requirement.** DLO-Lab explicitly
  penalizes the gripper approaching the target directly (`−Σ max(0, ε − d(p_ef, p_obj))`), which
  makes the task genuinely DLO-mediated *and* guarantees the flat region — the concept is partly
  self-inflicted by good task design.
- **Repaired by construction.** DaXBench's *local action adjustment* forcibly snaps the gripper
  onto the object each step so contact is never absent; this removes inaccessibility at the price
  of restricting the action space. Not applicable when the task requires a tool.
- **Partial accessibility in free-flight dynamics.** When the reward depends on the *manipulated
  body's own* state (rope tip position) rather than a third object, the gradient survives through
  free flight, and only a terminal strike/contact event reintroduces the discontinuity. This is
  the regime where first-order polish remains defensible.

## Comparison

Against the standard reading of differentiable-physics benchmarks: DaXBench concluded that
gradient-based methods help on some DOM tasks and hurt on others, attributing the failures mostly
to gradient *quality*. DLO-Lab sharpens this into a structural claim by exhibiting three tasks
where the gradient is not low-quality but absent, and reporting that GD lands exactly on the
do-nothing reward floor (Slingshot 6.90, 0% success) while CMA-ES reaches 11.07 / 93%.

Against sparse-reward RL: superficially similar ("no learning signal until something happens"),
but the mechanisms differ. Sparse reward is a *credit assignment* problem solvable with
exploration bonuses, hindsight relabeling, or curricula. Gradient inaccessibility is a *local
geometry* problem: the reward may be dense in time yet constant in the action neighbourhood.

## Known limitations

- The concept is asserted from benchmark evidence, not proved for a class of tasks. It has no
  agreed diagnostic — nobody reports "fraction of the action neighbourhood with `‖∂r/∂a‖ > 0`",
  which would make it measurable rather than anecdotal.
- Reward shaping can dissolve it (add a rope-to-object distance term, as DLO-Lab does in
  *Gathering*), so it is a property of the *reward*, not of the physics — yet it is usually
  discussed as if it were a simulator property.
- It says nothing about whether gradients help once contact exists; ruggedness is a separate
  failure that the same tasks also exhibit.

## Open problems

- A cheap pre-flight test that classifies a task as gradient-accessible before spending a
  training run on GD.
- Agent- or heuristic-planned initialization trajectories that establish contact and hand a
  non-zero gradient to the optimizer — proposed as future work in
  [[dlo-lab-benchmarking-deformable-linear-object]], unimplemented.
- Hybrid estimators that interpolate between zeroth-order sampling and first-order analytic
  gradients based on a measured local flatness statistic.

## Relationship to foundations

Sits on [[trajectory-optimization]] and [[optimization]] (it is a statement about the objective's
local geometry) and on [[contact-rich-manipulation]] (contact is what switches the gradient on).
The remedy space overlaps [[cross-entropy-method]] and other population-based zeroth-order search.

## Realized by

- [[dlo-agent]] — its task-decomposition capability is partly a workaround: subtask rewards
  written per phase keep a dense, locally informative landscape where a single monolithic reward
  would be flat.

## My understanding

This is the single most decision-relevant idea in DLO-Lab for the rope-swing project, and it cuts
in a direction that is easy to get backwards.

The gated Stage-D item "differentiable-sim gradient polish"
([[sim-stage-d-gated-extensions]] D5) assumes that if a simulator exposes gradients, a first-order
pass will refine a selected candidate. DLO-Lab's Slingshot result is the closest published test of
that assumption on a rope-launches-projectile task, and gradient descent scored the *do-nothing
floor* — not a modest improvement, not a wash, but complete failure, while an 18-parameter CMA-ES
search hit 93%.

The reason the rope-swing task is not automatically doomed is the fourth variant above: the
project's cost is a function of the **rope's own tip**, so tip-position error is a continuous
function of the handle trajectory throughout the swing, and the gradient exists before any
terminal event. That is a materially better position than Slingshot, whose reward lived on a cube
the rope had not touched.

But the project's cost is not purely tip position. Arrival-*direction* error evaluated at the
interpolated closest-approach instant of a designated pass is a soft-min over a trajectory — a
selection operator whose gradient switches which timestep it reads as candidates move, which is
the ruggedness failure rather than the inaccessibility failure. So the honest reading is: D5 is
gradient-*accessible* but not gradient-*friendly*. Scope it as local polish inside a basin that
sampling already found, keep [[sim-verified-best-of-n-selection]] as the primary mechanism, and
require the D5 gate to show a measured improvement over CMA-ES refinement of the same candidate
before it earns any place in the pipeline.
