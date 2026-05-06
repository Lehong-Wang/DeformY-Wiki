---
title: "Real2Sim2Real reaches 8–14% median tip-error on planar robot casting"
slug: "real2sim2real-prc-tip-error-8-14-percent"
status: supported
confidence: 0.8
tags: [DLO, sim-to-real, real2sim2real, robot-casting, free-end-cable, supervised-learning]
domain: "Robotics"
source_papers: ["[[planar-robot-casting-real2sim2real-self-supervised]]"]
evidence:
  - source: "[[planar-robot-casting-real2sim2real-self-supervised]]"
    type: supports
    strength: strong
    detail: "Per-cable median tip-to-target error of 8% (Cable 1, 0.63 m paracord), 12% (Cable 2, 0.65 m nylon), and 14% (Cable 3, 0.65 m jump rope) of cable length on a UR5 with 16 targets × 5 trials each (75 trials per cable). The pipeline uses |D_real| ≈ 522 trajectories, DE-tuned Isaac Gym segmented (FleX) simulator, |D_sim| ≈ 21,450, and a forward dynamics NN trained on D_real ∪ D_sim with D_real upsampled to 30–40% and weighted higher in the loss. Beats analytic Cast-and-Pull (61%), Gaussian Process on D_real (27%), pure-real NN (15%), and pure-sim NN (14%) on Cable 1; the mixed-data NN is the only policy with median below 15% on every cable. Targets may lie outside the UR5's own reachable workspace (cable extends reach)."
conditions: "Planar 2D workspace (2.45 m × 1.55 m masonite), UR5 with single open-loop dynamic motion (~2 s execution + ~20 s reset), 5-DoF parameterized action a = (θ₁, θ₂, r₂, α, v_max) with v_max ≤ 2.5 m/s, three specific cables (paracord 8 g / nylon 50 g / jump rope 45 g), Isaac Gym segmented FleX simulator, DE tuning with |D_tune|=20, mixed-data NN with 30–40% real-data upsampling. The 8–14% range is per-cable median; max errors reach 105% on Cable 1 for at least one outlier target."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

The Real2Sim2Real pipeline — autonomous physical data collection, Differential Evolution tuning of a parameterized cable simulator, then mixed-data supervised training of a forward dynamics model — yields per-cable **median tip-to-target error of 8% / 12% / 14% of cable length** on the Planar Robot Casting task across three cables of differing stiffness, mass, and friction. This holds on a UR5 with single-step open-loop dynamic actions, including targets outside the robot's own reachable workspace.

## Evidence summary

The source paper runs 16 targets × 5 trials = 75 physical rollouts per cable, with the same NN architecture and the same R2S2R pipeline applied per cable. The reported medians (8% / 12% / 14% of cable length) are accompanied by tight first/third quartiles for cables 2 and 3 (Q1=8/9%, Q3=16/23%) and a wider third-quartile but still single-target maximum of 105% on Cable 1.

Crucially, the **R2S2R policy** (median 8% on Cable 1) cleanly outperforms the policy trained on $\dreal$ alone (median 15%) and on $\dsim$ alone (median 14%), supporting the central methodological claim — *mixing helps*, simulator tuning alone is not enough. The $\pisim$ policy has competitive median but a 115% max error on a single target, which $\pitune$ partially clamps (max 105%) — suggesting the value of mixing is most visible in the tail.

## Conditions and scope

The claim holds **only** under the conditions listed above: planar 2D workspace, single open-loop dynamic action, Isaac Gym segmented (FleX) simulator, DE tuning, mixed-data weighting, UR5, three specific cables. The 8–14% range is the *median*; per-target outliers can reach a full cable length.

## Counter-evidence

- **Tail outliers** within the same paper: $\pisim$ max = 115%, $\pitune$ max = 105% on Cable 1. The pipeline does not eliminate large-error rollouts, just shifts the median.
- **Per-cable training** is required — the paper does not demonstrate that one R2S2R-tuned model transfers across cables, so 8/12/14% is the *best-case-per-cable* number, not a generalization bound.
- The claim does not extend to **3D** (Spatial Robot Casting), **closed-loop** policies, or **fixed-end** cables — those settings have not been tested here.
- The successor paper from the same group (Wang et al. 2024, "Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables") extends this pipeline to longer cables and re-confirms the recipe, providing an additional supporting datapoint not yet ingested.

## Linked ideas

(none yet — DeformY's natural follow-up is a closed-loop, full-3D extension that aims to inherit this median while reducing the tail and removing the per-cable retraining cost.)

## Open questions

- Does the same 8–14% range survive when the policy must also generalize across cables?
- Is the 105–115% tail intrinsic to open-loop control, or does it disappear under closed-loop visuomotor policies?
- How does this baseline compare against (a) IRP-style residual policies, (b) a Cosserat rod simulator instead of Isaac Gym FleX segmented, (c) differentiable simulator tuning instead of DE?
