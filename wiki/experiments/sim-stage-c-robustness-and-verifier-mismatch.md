---
title: "Sim Stage C — Robustness, Robust Atlas, and the Verifier-Mismatch Ranking Test"
slug: sim-stage-c-robustness-and-verifier-mismatch
status: planned
linked_idea: "[[direction-conditioned-open-loop-rope-tip-targeting]]"
evaluates_methods:
  - "[[sim-verified-best-of-n-selection]]"
  - "[[direction-reachability-atlas]]"
hypothesis: "Candidate ranking survives model error of the size a few-minute calibration actually leaves behind: top-N candidates ranked in a perturbed simulator and executed in the nominal simulator retain most of their nominal success, and the degradation is a graceful function of verifier-error magnitude. Margin-aware selection (expected cost under ~100 perturbed clones) beats nominal selection."
tags: [robustness, verifier-mismatch, ranking-robustness, calibration-residual, margin-aware-selection, robust-atlas, handle-tracking-error, sim-to-real, DLO, rope]
setup:
  model: "Stage-B winning amortizer, frozen"
  dataset: "Stage-B held-out eval goals; ~100 perturbed rope clones per candidate; simulated-calibration residual distribution"
  hardware: "Remote sim server, RTX 4090"
  framework: "Isaac Lab + Stable Cosserat Rods (DeformX/Cosserat-Rod-Sim-CUDA); PyTorch"
metrics: [top-n-success-vs-verifier-error-magnitude, ranking-correlation-perturbed-vs-nominal, margin-aware-vs-nominal-selection-success, error-cdf-per-component, robust-reachable-fraction, calibration-parameter-residual]
baseline: "Nominal-model selection (Stage-B verified top-N) as the reference; ad-hoc ±% perturbations as the naive comparison for calibration-consistent perturbations"
date_planned: 2026-07-29
estimated_hours: 40
---

## Objective

Test **the single load-bearing assumption of the entire project**, and test it in simulation
where failing is cheap.

On the real robot, top-N selection will be ranked by a **few-minute-calibrated** simulator, not
by the true dynamics. Whether a slightly-wrong model still *ranks* candidates correctly is
untested, and it is the one place the campaign could silently build toward a dead end: every
Stage-B verified number depends on it, and none of them measures it. Absolute sim accuracy is
not what is needed — **ranking robustness** is.

External evidence that this bites: Wiggle&Go reports 3.55 cm real 3D striking degrading to
**15.34 cm** under poor parameter fidelity
([[wiggle-go-system-identification-zero-shot]]).

## Setup

Freeze the Stage-B winner. Perturbation magnitudes are **not** ad-hoc ±%: they are derived from
a **simulated calibration procedure** — draw a randomized rope, run the intended few-minute
calibration against it in sim, measure the parameter residual, and use *that* distribution. A
useful side effect is that this forces the calibration procedure to be designed on paper now,
while it is still cheap to change.

Known simulator limits that feed the perturbation model (from the simulator repo's own
sim-to-real list): **linear + isotropic air drag** (its own #1 item), rigid tube, no rope
self-collisions.

## Procedure

1. **Verifier-mismatch ranking test.** Rank the N candidates in a simulator perturbed by a
   plausible calibration residual, then execute the selected candidate in the **nominal**
   simulator — and vice versa. Report top-N success as a function of verifier-error magnitude,
   and the rank correlation between perturbed and nominal orderings.
2. **Margin-aware selection vs nominal selection.** Score candidates by expected cost under
   ~100 perturbed clones instead of nominal cost. This converges with robust verification: score
   under K rope-parameter perturbations and pick the most robust, not the nominally best.
3. **Error CDFs per component** — position and direction separately, never pooled.
4. **Robust atlas.** Where is (p, d̂) striking *reliably* possible open-loop, as opposed to
   possible once? The robust layer of [[direction-reachability-atlas]].
5. **Handle-trajectory perturbations.** Sim-to-real is not only a rope problem: the rope's true
   input is the **achieved** handle motion, not the commanded one, and tracking error at the
   dynamic envelope is a gap no amount of rope calibration absorbs. Include commanded-vs-achieved
   arm-tracking error in the perturbation model, and restrict the action envelope conservatively
   inside the real arm's demonstrated tracking envelope (measured by the one-day rope-free check
   on the real arm, allowed under the brief's assets).

## Results

Not yet run.

## Analysis

Pending. Reporting rule: **robust-core-only claims.** If direction margins collapse under
perturbation, the honest output is a smaller robust atlas, not a re-tuned threshold. Open-loop
fragility is a finding about the task, and reporting it as such is the point of having the
robust layer at all.

## Idea updates

Pending. This stage resolves sub-hypothesis H5 of
[[direction-conditioned-open-loop-rope-tip-targeting]] — the one whose failure invalidates the
deployment chain regardless of how well Stages A and B went.

If ranking robustness fails, the response is not to abandon verification but to change what is
verified: robust scoring over perturbed clones, a second independent verifier (DLO-Lab's
released differentiable DLO sim is a candidate), or DA-MMP-style outcome-conditioning — condition
the flow on a handful of observed real outcomes, the cheapest published sim-to-real correction
for exactly this pipeline. An IRP-style residual tip-dynamics model
([[iterative-residual-policy]], [[delta-dynamics-network]]) also re-enters as a live candidate
here: forward models were rejected **for the sim phase only**, not as doctrine.

## Follow-up

[[sim-stage-d-gated-extensions]] runs by results. The real-robot campaign is out of this
campaign's scope, but this stage is what decides whether it is worth starting: the artifacts
that carry forward are the calibration procedure design, the residual distribution, the robust
atlas, and the tracking envelope.
