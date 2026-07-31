---
title: "Trajectory Manifold Optimization"
aliases: ["TMO", "constraint-aware manifold fine-tuning", "generator fine-tuning under differentiable constraints", "amortized constraint satisfaction", "task-loss fine-tuning of a motion generator"]
tags: [kinodynamic-constraints, trajectory-generation, motion-manifold, amortized-planning, constraint-satisfaction, fine-tuning, differentiable-constraints, task-conditioned-generation]
maturity: emerging
definition: "Fine-tuning a trained conditional motion generator against a differentiable task objective plus a constraint-violation penalty, with the conditioning variable sampled over the whole continuous task space, so that constraint and task satisfaction are amortized into the generator itself rather than checked per-sample at plan time."
key_papers:
- "[[differentiable-motion-manifold-primitives-reactive-motion]]"
first_introduced: "2024"
date_updated: 2026-07-30
related_concepts:
- "[[motion-manifold-primitives]]"
parent_topic: compact-action-parameterization
---

## Definition

A generative motion model fitted to a dataset of trajectories reproduces the *data*, not the
*specification*. **Trajectory Manifold Optimization (TMO)** is the second training stage that
closes that gap: with the data-fitting stage complete, the generator's output map is fine-tuned
to minimize

$$\mathbb{E}_{t\sim U[0,T],\ \tau\sim U(\mathcal{T}),\ z\sim p(z|\tau)}\Big[\,J\big(\hat q(z,t);\tau\big) \;+\; W^\top\,\mathrm{ReLU}\big(C(\hat q,\dot{\hat q},\ddot{\hat q},\dddot{\hat q})\big)^2\,\Big] \;+\; w_{\rm recon}\mathcal{L}_{\rm recon},$$

where $J$ is the task objective, $C \le 0$ the constraint vector, and $\mathcal{L}_{\rm recon}$
an anchor that keeps the generator near its fitted manifold. Three properties define the
concept and each one is load-bearing:

1. **The constraints enter the loss, not a post-hoc filter.** This requires the generated
   trajectory to be a *differentiable* function of the model parameters at every derivative
   order the constraints touch (velocity, acceleration, jerk, torque via inverse dynamics).
2. **The conditioning variable is sampled over its full continuous domain**, not over the finite
   set of task parameters that appear in the training data. This is what converts the procedure
   from "polish the training points" into "generalize to unseen tasks".
3. **Only the generator's output map is updated.** Encoders, latent densities, and ODE-based
   samplers are frozen, because backpropagating through sampled latents and an ODE solver is
   prohibitively expensive and unnecessary — moving the decoder alone moves the reachable set.

## Intuition

Two ways to get feasible motions out of a learned generator: filter its samples, or move the
generator. Filtering (rejection sampling, verification, guidance at inference) is cheap to
implement and costs compute at every deployment. TMO pays once, offline, and reshapes the set
of motions the generator can emit at all — an *amortization* of the constraint-satisfaction
problem in the same sense that the generator itself is an amortization of the planning problem.

The reason it is not simply "add a penalty to the training loss" is the sampling distribution.
The reconstruction loss is an expectation over the *dataset*; the TMO loss is an expectation
over the *specification* — every task parameter in the domain, every latent the density
actually emits, every time along the trajectory. The generator is being trained against the
problem statement, not against solutions to a finite subset of it.

## Variants

- **Decoder-only fine-tuning under a frozen latent flow** ([[differentiable-motion-manifold-primitives-reactive-motion]],
  the originating instance): manifold decoder $f_\beta(z,t)$ fine-tuned while encoder and
  task-conditioned flow are frozen.
- **Filter-only alternative** (rejection sampling / feasibility verification at deployment): the
  degenerate case with no fine-tuning. In practice the two compose — the originating paper needs
  both, TMO taking success 17.5% → 95.8% and rejection sampling closing 95.8% → 100%.
- **Guidance-at-inference alternative** (classifier / cost guidance in diffusion planners, cf.
  [[planning-as-diffusion]]): steers each sample at generation time instead of moving the model;
  pays per sample, but needs no retraining when the constraints change.
- **Architecture-agnostic form**: nothing in the loss requires an autoencoder. Any conditional
  generator whose output is analytically differentiable in time — including a flow-matching
  model over fixed smooth-basis curve parameters — supports the identical objective.

## Comparison

- vs **plain data fitting**: fitting reproduces the trajectory-optimization solutions it was
  shown; those solutions were feasible, but their reconstructions are not. In the originating
  experiment, discrete-time models fitted to feasible data scored **0.0%** on velocity,
  acceleration, jerk and torque limits.
- vs **rejection sampling / test-time verification** ([[sim-verified-best-of-n-selection]]):
  complementary, not competing. TMO shrinks the infeasible mass so that a fixed sampling budget
  contains feasible candidates; verification guarantees the executed one is feasible. TMO
  without verification leaves residual violations (velocity limits at 80–93%); verification
  without TMO has almost nothing feasible to select from.
- vs **constrained trajectory optimization**: TMO solves the same constrained problem, but once,
  in weight space, for all task parameters simultaneously — trading a per-query 10–3000 s solve
  for a 0.012 s forward pass, at the cost of a soft penalty rather than a feasibility
  certificate.

## Known limitations

- **Soft penalty, not a guarantee.** $\mathrm{ReLU}(C)^2$ is a penalty; residual violations
  survive and concentrate on the tightest constraint (joint-velocity limits in the originating
  paper). A downstream verifier is still required for a hard guarantee.
- **Requires a differentiable simulator/constraint stack.** Forward kinematics, inverse dynamics
  and collision margins all have to be autodiff-able in the training loop.
- **Weight vector $W$ is a tuning surface.** Relative weighting of $J$ against each constraint
  family, and of $w_{\rm recon}$ against both, is unreported and likely sensitive.
- **The frozen encoder can drift out of correspondence.** After the decoder moves, the frozen
  encoder no longer inverts it; latent semantics learned in stage 2 are no longer guaranteed.
- **Cost scales with the constraint stack**, since the penalty is evaluated at sampled times over
  sampled latents over sampled tasks at every training step.

## Open problems

- No ablation exists separating "fine-tune the generator against constraints" from "have a
  manifold at all" — the concept's value independent of its originating architecture is untested.
- How to schedule $w_{\rm recon}$: too high anchors the generator to infeasible data, too low
  lets it collapse away from the demonstrated diversity.
- Whether TMO can be run *online* against a changed constraint set (new obstacle, degraded
  actuator) at a cost below re-solving, or whether it is inherently an offline stage.
- Interaction with latent geometry: does constraint-driven decoder movement destroy the isometry
  properties that [[motion-manifold-primitives]] variants work to establish?

## Relationship to foundations

TMO is [[trajectory-optimization]] moved from per-query variable space into a generator's weight
space: the same objective and the same constraints, but solved once, amortized over the task
distribution, with penalty methods ([[optimization]]) standing in for hard constraint handling
because the optimizer is now stochastic gradient descent ([[gradient-descent]]) rather than SQP.

## Realized by

- [[dmmp-differentiable-motion-manifold-primitives]] — DMMP/DMMFP + TMO + rejection sampling: the
  four-stage pipeline in which TMO is stage 4, on 7-DoF dynamic throwing.

## My understanding

For the rope-swing project this is the single transferable component of the whole
manifold/autoencoder line, and it is transferable precisely because it does not need the
manifold. The project's amortizer ([[conditional-flow-matching-motion-parameters]]) emits
parameters of an analytic smooth basis ([[smooth-basis-swing-parameterization]]), so
$q(t; w)$ is differentiable to arbitrary order in $t$ and differentiable in $w$ — exactly the
condition TMO needs. The recipe drops in as an extra training stage: after flow-matching on the
hindsight-relabeled pool, keep sampling $w \sim p(w \mid g)$ with $g$ drawn **uniformly over the
pre-registered task box** and penalize jerk, joint-limit and velocity violations of the decoded
swing. The originating paper's generalization numbers are the argument: its data-fitted baseline
fell from 77.4% on seen task parameters to 15.0% on unseen ones, while the TMO-fine-tuned model
held 95.8% → 94.1%. That is the same seen/unseen collapse that uniform eval sampling in
[[sim-stage-b-amortization-shootout]] is designed to expose — and TMO is a fix for it, not just
a diagnostic.

The honest caveat: the rope's own tip trajectory is *not* a differentiable function of $w$ in
closed form (the simulator stands between them), so only the **arm-side** constraints — jerk,
joint limits, velocity/acceleration limits, workspace bounds — are directly TMO-able. Tip-side
objectives would need a differentiable rope simulator, which is a different and much larger
commitment.
