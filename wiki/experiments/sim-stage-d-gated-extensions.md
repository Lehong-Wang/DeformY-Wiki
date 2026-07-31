---
title: "Sim Stage D — Gated Extensions (Speed Goal, Mirror Symmetry, Multi-Rope, Wind-Up, Gradients)"
slug: sim-stage-d-gated-extensions
status: planned
linked_idea: "[[direction-conditioned-open-loop-rope-tip-targeting]]"
evaluates_methods:
  - "[[smooth-basis-swing-parameterization]]"
  - "[[conditional-flow-matching-motion-parameters]]"
  - "[[direction-reachability-atlas]]"
hypothesis: "Each extension is a strict addition to the base pipeline, requiring no re-architecture of Stages 0–C. Specifically: D1 impact speed is learnable as a 6th goal dimension in the same conditional model; D2 the wall-mounted rig is left-right mirror-symmetric, making x → −x a 2× exact data augmentation; D3 rope-context conditioning generalizes the amortizer across ropes; D4 longer energy pumping lifts range/direction saturation if the atlas shows it; D5 differentiable-sim gradients polish candidates where gradients are exposed."
tags: [extensions, gated, speed-goal, mirror-symmetry, multi-rope, privileged-conditioning, wind-up, differentiable-simulation, RMA, DLO, rope]
setup:
  model: "Stage-B winning amortizer, extended per sub-experiment"
  dataset: "Stage-A pool re-instantiated at ~3 rope settings for D3; mirror-folded pool for D2"
  hardware: "Remote sim server, RTX 4090"
  framework: "Isaac Lab + Stable Cosserat Rods (DeformX/Cosserat-Rod-Sim-CUDA); PyTorch"
metrics: [speed-goal-success, mirror-augmentation-exactness, cross-rope-success, atlas-coverage-vs-windup-duration, gradient-polish-improvement]
baseline: "Stage-B/C results without the extension, per sub-experiment"
date_planned: 2026-07-29
---

## Objective

Hold the extensions that are deliberately **out** of the base experiment, so that "not built
yet" never gets confused with "not thought through". Each is gated on a specific trigger, and
each is an addition rather than a redesign — nothing built in Stages 0–C becomes throwaway if
any of these run.

## Setup

Gates are listed per sub-experiment below. None of these should start before
[[sim-stage-c-robustness-and-verifier-mismatch]] reports, except D2, which is a cheap
correctness check that Stage A's data pipeline depends on if mirror augmentation is used at all.

## Procedure

**D1 — Speed as a goal component.** Gate: an application needs impact speed. Adds a 6th goal
dimension. Achieved speed is already logged and stratified everywhere from Stage A onward, and
a hard v_min constraint is already available as a knob, so the data to fit this exists before
the experiment does.

**D2 — Mirror-symmetry validation.** Gate: run before relying on the augmentation. Is
x → −x across the mount's vertical plane *exact* on this rig? This matters because the
azimuth-equivariance reduction was **retracted** on 2026-07-29 — the rig is wall-mounted (base
at z = 1.7 m, rotated −90° about X), so the base joint axis is horizontal and rotating it does
not commute with gravity. A single left-right mirror is what may survive: a 2× data
augmentation, not a dimension reduction. Cost of it failing: 2× data, nothing else.

**D3 — Multi-rope privileged conditioning.** Gate: base pipeline validated. Scaffolds the future
real-robot calibration campaign — the amortizer gains a rope-context input, privileged rope
parameters in simulation and a calibration-inferred embedding on the real robot, in the
[[rma-rapid-motor-adaptation]] / [[amortized-context-encoder-adaptation]] style. Same model
class, extended condition. Also produces the atlas at ~3 rope settings, which is what upgrades
[[direction-reachability-atlas]] from *methodology + one instance* to a claim with external
validity. Rope properties are parameterized in all sweep/relabel/atlas code from day 1
specifically so this needs no rewrite.

**D4 — Deep wind-up.** Gate: the atlas shows range or direction saturation that longer energy
pumping could plausibly lift. Note the related deferred design: feedback on the **slow wind-up
phase only** — stabilizing the pre-strike initial condition as a [[limit-cycle-control]]
problem, then committing to an open-loop strike — is
the *only* closed-loop variant the project will ever consider, and it is a non-core fallback —
recorded in the 2026-06-16 control-regime decision, not scheduled here.

**D5 — Differentiable-sim gradient polish.** Gate: the remote environment exposes gradients.
The simulator of record does not, currently.
[[dlo-lab-benchmarking-deformable-linear-object]] (ICML 2026, Apache-2.0, code released) does —
Taichi autodiff over the whole rod solver including contact, with FluidLab-style CPU gradient
checkpointing making backward memory independent of horizon.

**But it is also the strongest argument for keeping D5 gated (evidence added 2026-07-30).** On
its own benchmark, analytic-gradient descent hit the **do-nothing floor with 0% success on all
three tool-use tasks** (Gathering, Lifting, Slingshot) because ∂r/∂a ≡ 0 before contact, while
an 18-parameter CMA-ES search reached 93% on Slingshot. Overall: CMA-ES 86.6% average success
vs SAPO 35.0%, PPO 29.3%, gradient descent 25.0%. The concept is recorded as
[[gradient-inaccessibility-contact-mediated-manipulation]].

The rope-swing task is better placed than those three — the tip position is continuous in
handle motion throughout free flight, so there is no pre-contact dead zone — but the
closest-approach soft-min direction term reintroduces landscape ruggedness. **Revised gate:**
scope D5 to polishing a CMA-ES-selected candidate *inside an already-smooth basin*, and require
a measured win over CMA-ES refinement of the same candidate before it enters the pipeline.

## Results

Not yet run.

## Analysis

Pending per sub-experiment.

## Idea updates

Pending. D3 is the bridge from the sim campaign in
[[direction-conditioned-open-loop-rope-tip-targeting]] to its actual final goal — real robot,
any rope after a one-time few-minute calibration, open-loop zero-shot on (position, direction).
D2's outcome is the only one that can invalidate an assumption already baked into Stage A rather
than merely adding capability.

## Follow-up

Real-robot campaign, out of scope for the pure-simulation plan. Two items to carry into it:
DA-MMP's outcome-conditioning trick (condition the flow on a handful of observed real outcomes)
as the cheapest published sim-to-real correction for this pipeline, and the adaptive-N idea —
candidate count can vary per goal rather than being fixed.
