---
title: "Sim Stage 0 — Harness, Locked Protocol, and Smooth-Regime Check"
slug: sim-stage-0-harness
status: planned
linked_idea: "[[direction-conditioned-open-loop-rope-tip-targeting]]"
evaluates_methods:
  - "[[smooth-basis-swing-parameterization]]"
hypothesis: "The env interface + smooth-basis decoder + cost function sustain ≥10³ rollouts/s aggregate with deterministic replay, the direction cost term is visibly optimizable, and in the intended smooth-swing regime (tip-path curvature radius r ≈ 1.5–3 m at v ≈ 5 m/s) 5–20 ms of open-loop timing jitter costs only ~1–3° of arrival direction."
tags: [harness, evaluation-protocol, smooth-regime, jitter-sensitivity, cost-landscape, determinism, DLO, rope, simulation]
setup:
  model: "No learned model — decoder + cost only"
  dataset: "~10² diverse min-jerk rollouts for the smooth-regime probe"
  hardware: "Remote sim server, RTX 4090"
  framework: "Isaac Lab + Stable Cosserat Rods (DeformX/Cosserat-Rod-Sim-CUDA), 60 Hz control, 100 substeps/frame; PyTorch"
metrics: [rollouts-per-second, replay-determinism, tip-path-curvature-radius, direction-rotation-rate, direction-spread-under-jitter, cost-landscape-slice-structure]
baseline: "None — this stage establishes the measurement apparatus"
date_planned: 2026-07-29
estimated_hours: 40
---

## Objective

Build the measurement apparatus and **lock the evaluation protocol before any data exists**,
then cheaply verify the one physical assumption the whole direction goal rests on: that the
intended operating regime is benign with respect to open-loop timing jitter.

Ordering is a review-mandated constraint, not a preference. The success predicate and the task
box (plan §6.6) are locked here, **before the first sweep** — a predicate chosen after seeing
data can always be chosen to flatter it.

## Setup

Thin rollout interface: `rollout(W: [B, dim_w]) -> tip_traj: [B, T, 3]`, plus tip **state
velocities** (not finite-differenced positions — exact, free, and robust if fast local events
occur). Closest approach between frames is interpolated. The simulator's native interface is
per-step RL (RSL-RL PPO, joint deltas); the harness bypasses the policy loop and drives joint
target sequences directly via the `play.py --export` path.

Decoders: Family A (joint-space via-points, K ∈ {4, 6}) and Family B (physics-informed strike
frame) from [[smooth-basis-swing-parameterization]].

**Cost(w; g)** = soft-min over t of ‖y(t) − p*‖ (temperature-annealed)
+ λ_dir · Σ_t ω(t) · (1 − v̂(t)·d̂*) with proximity weights ω(t) ∝ softmax(−‖y(t) − p*‖/T)
+ small jerk penalty. The proximity weighting and temperature annealing are deliberate
[[reward-shaping]] — the raw predicate (did the tip pass within 5 cm along d̂?) has no usable
gradient or ranking signal. λ_dir is active from day 1; ramping it during iterative
optimization is an optimizer trick, not a scope change.

## Procedure

**Locked here, before any sweep:**

1. **Success predicate.** Direction is evaluated at the **interpolated closest-approach instant
   only**, on **one designated pass** per rollout. Near-passes are reported as a diagnostic and
   never cherry-picked. Success = position ∧ direction thresholds at that instant, stratified
   by tip speed. Longer or multi-pass trajectories get no extra chances.
2. **Canonical exchange rate.** One rate — 5 cm ≡ 15°, from the headline metric — drives the
   CVT metric, the default λ_dir, and the goal-space distance used in balancing. Three places
   that would otherwise each pick their own.
3. **Task box** per plan §6.6 (dome 1.5–2.4 m, uniform-S² directions, swept-segment hit
   detection, 3 s episodes at 60 Hz, primary endpoint 5 cm ∧ 30° ∧ ≥ 0.3 m/s), plus all
   committed secondary endpoints.
4. **Degeneracy guard.** Direction is credited only when tip speed at approach ≥ v_ε. Achieved
   speed is always logged.

**E0 probes:**

5. Throughput and determinism: replay identical W twice, bit-compare.
6. **Cost-landscape slices** of position *and* direction components vs w, near random points
   and near elite points — is the direction term structured enough to optimize at all?
7. **Smooth-regime check.** Along ~10² diverse min-jerk rollouts, measure the distribution of
   local tip-path curvature radius and direction rotation rate v/r; inject ±5–20 ms timing
   jitter and measure the resulting arrival-direction spread.

## Results

Not yet run.

## Analysis

Design expectation for probe 7: r ≈ 1.5–3 m at v ≈ 5 m/s gives ≈ 2–3 rad/s ≈ 140°/s, so
5–20 ms of jitter costs ~1–3°, and 60 Hz output resolves direction to ~1–2°/frame. The probe
**verifies** this rather than assuming it, and maps where along a trajectory — if anywhere —
the rope's energy focusing produces fast local events that break it. Tight-curvature
whip-crack dynamics (r ~ 0.2–0.5 m, where jitter would cost tens of degrees) are outside the
project's declared regime.

The caveat driving the probe: **smooth commands do not guarantee smooth tip paths.** The rope
is underactuated and focuses energy, so local curvature along the tip path under min-jerk
commands is an empirical distribution, not a design guarantee.

## Idea updates

Pending. If probe 7 fails — i.e. tip paths under smooth commands still contain
tight-curvature events — the operating envelope must be restricted **before Stage A**, and the
"precision is not the risk" premise of
[[direction-conditioned-open-loop-rope-tip-targeting]] needs re-examination.

## Follow-up

**Exit criteria (all required to proceed):**

- ≥ 10³ rollouts/s aggregate;
- deterministic replay;
- direction term visibly optimizable in the landscape slices;
- smooth-regime check consistent with the ~1–3° jitter floor.

On pass → [[sim-stage-a-atlas-and-data-factory]]. On failure of the last criterion → flag and
restrict the operating envelope first.

Additional one-day check allowed under the brief's assets and needed before Stage C: a
rope-free **arm-tracking envelope** measurement on the real robot, to bound the action space
conservatively inside the arm's demonstrated tracking range.
