---
title: "Sim Stage B — Amortization Shootout and Evaluation Ladder"
slug: sim-stage-b-amortization-shootout
status: planned
linked_idea: "[[direction-conditioned-open-loop-rope-tip-targeting]]"
evaluates_methods:
  - "[[conditional-flow-matching-motion-parameters]]"
  - "[[sim-verified-best-of-n-selection]]"
  - "[[per-timestep-hindsight-relabeling]]"
hypothesis: "Trained on the balanced relabeled pool, a conditional flow-matching amortizer reaches ≥85% success at 5 cm ∧ 15° under sim-verified top-N (N = 8/64) and ≥65% blind top-1, on rollout-level held-out goals sampled uniformly over the pre-registered task box; and direction conditioning rehabilitates deterministic regression into a competitive arm, while pool nearest-neighbour lookup degrades in 5D."
tags: [amortization, shootout, flow-matching, regression, nearest-neighbour, motion-manifold, GCSL, evaluation-ladder, holdout, wilson-intervals, DLO, rope]
setup:
  model: "B1 pool NN + 1-step CMA-ES polish; B2 deterministic MLP regression; B3 conditional flow matching; B4 conditional manifold decoder f(z, g), z ∈ ℝ²–³; B5 GCSL loop on the winner"
  dataset: "Stage-A balanced relabeled pool + A3 coverage-directed elites; rollout-level train/eval split"
  hardware: "Remote sim server, RTX 4090"
  framework: "PyTorch; torchcfm (MIT) + ALRhub/MP_PyTorch; movement-primitive-diffusion as architectural reference; pycma for oracle probes"
metrics: [success-at-5cm-and-15deg, success-grid-position-x-direction, blind-top1-success, verified-top8-success, verified-top64-success, position-median-p90, direction-median-p90, achieved-speed-distribution, success-vs-distance-to-nearest-training-pair, amortization-gap-vs-oracle, wilson-interval-width]
baseline: "B1 pool nearest-neighbour as sanity floor; CMA-ES oracle as upper bound; DIDP 84.3% @ 5 cm (position-marginal, sim-only) as the external bridge"
date_planned: 2026-07-29
estimated_hours: 80
---

## Objective

Decide which amortizer to carry forward, on a protocol hardened against the three stacked
circularities that could otherwise produce impressive-but-hollow numbers. All arms train on the
same pool and are evaluated on the same held-out goals.

## Setup

**Hardened evaluation protocol** — this is the part that matters more than the arms:

- **Rollout-level holdout.** A rollout and *all* its relabeled pairs live in one split.
  Cell-level holdout leaks near-identical neighbours into training.
- **Pre-registered task box.** Locked in Stage 0, *before* Stage A ran (plan §6.6). Success is
  reported both on it and on the atlas-derived reachable set. A box declared after seeing the
  atlas can always be drawn around the successes.
- **Uniform eval sampling** over the pre-registered box — not from the natural achieved
  measure. Additionally report success as a function of **distance-to-nearest-training-pair**,
  so memorization-plus-smoothing is visible rather than hidden.
- **Statistical floor.** Fixed eval-set size with Wilson confidence intervals; shootout
  decisions require non-overlapping intervals or a paired test on shared goals.

**Metrics grid, never a single scalar:** position ≤ {10, 5, 2} cm × direction ≤ {30, 15, 7.5}°
jointly, headline 5 cm ∧ 15°, plus speed strata {0.3, 1, 3} m/s and direction bands against
radial (0–45 / 45–90 / 90–135 / 135–180°). Position-only marginals are reported as the bridge to
DIDP/IRP-era numbers. The two objectives trade off, and a scalar hides failure modes — Pareto
curves over λ_dir accompany the grid.

## Procedure

- **B1 — Pool nearest-neighbour (+1-step CMA-ES polish).** Sanity floor only; expected to degrade
  in 5D. If it *doesn't*, that is important news about the problem's effective dimension.
- **B2 — Deterministic regression** g → w (MLP). The arm that consequence #3 predicts will be
  rehabilitated by direction conditioning.
- **B3 — Conditional flow matching** g → w
  ([[conditional-flow-matching-motion-parameters]]). Hedge for whatever residual multimodality
  A4 found.
- **B4 — Conditional manifold** (DMMP / DA-MMP-style): decoder f(z, g) with small z (2–3) for
  residual diversity plus a latent density. **Necessary-condition check:** an *unconditional*
  manifold ([[mmp-parametric-curve-motion-manifold-primitives]]-style AE + p(z|g)) must have
  latent dim ≥ 4–5 just to span the canonical goal manifold — run as an ablation to show whether
  the conditional decoder's capacity placement actually matters.
- **B5 — GCSL loop on the winner.** Sample goals at high-error atlas cells → generate with noise
  → roll out → relabel → retrain.

**Evaluation ladder, run on every arm:** blind top-1 → sim-verified top-N (N = 8/64, the
legitimate offline deployment mode per [[sim-verified-best-of-n-selection]]) → + local CMA-ES
refinement (gap to oracle).

## Results

Not yet run.

## Analysis

Targets: sim-verified ≥ 85% at 5 cm ∧ 15° on atlas-covered goals; blind top-1 ≥ 65%;
position-marginal comparable to DIDP's 84.3% @ 5 cm as the external bridge
([[dynamic-manipulation-deformable-objects-3d-simulation]]).

Decision logic, from the base-experiment doc:

- **Verified success high** → pipeline validated; next spend goes to the coverage atlas and the
  learner shootout's remaining arms.
- **Verified fails even in-distribution** → the problem is the **action space or the cost**, not
  the learner. Go to cost-landscape slices and basis-count/duration sweeps *before* touching
  architecture.
- **Position works, direction fails** → first check whether the pool contains direction
  *diversity* per position region. If not, that is the coverage question arriving here instead
  of in Stage A; the levers are pre-swing posture freedom and swing-plane diversity, not model
  changes.
- **Blind ≪ verified** → acceptable (deployment verifies), but record it as a known dependence
  on calibrated-sim fidelity, because that dependence is what Stage C stress-tests.

The oracle probe keeps "the network is weak" and "the task is hard" from being conflated.

## Idea updates

Pending. This stage resolves sub-hypotheses H1 (supervision principle) and H4 (multimodality
collapse) of [[direction-conditioned-open-loop-rope-tip-targeting]]. If B2 matches B3 within
overlapping intervals, the generative head loses on complexity grounds — a legitimate and
publishable outcome, and one that would simplify the deployed system.

`target_venue` on the idea page is deliberately unset until this stage clears.

## Follow-up

On pass → [[sim-stage-c-robustness-and-verifier-mismatch]], which stress-tests the assumption
the ladder's verified numbers quietly depend on.

Every claim from here on is reported on the **success-vs-budget curve** (top-1 / 8 / 64 /
+CMA-ES) so it sits on a labeled point of the amortization–optimization continuum. The governing
per-goal compute ruling is still **proposed, awaiting user confirmation**.
