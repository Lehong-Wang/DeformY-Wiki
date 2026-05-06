---
title: "Free-end cable dynamic target reaching is harder than fixed-end on per-cable-length error"
slug: "free-end-cable-target-reaching-harder"
status: weakly_supported
confidence: 0.55
tags: [DLO, free-end-cable, fixed-end-cable, dynamic-manipulation, sim-to-real, robot-casting, error-comparison]
domain: "Robotics"
source_papers: ["[[self-supervised-learning-dynamic-planar-manipulation]]", "[[planar-robot-casting-real2sim2real-self-supervised]]"]
evidence:
  - source: "[[self-supervised-learning-dynamic-planar-manipulation]]"
    type: supports
    strength: moderate
    detail: "Free-end cable casting on a UR5 with the Real2Sim2Real pipeline (DE-tuned PyBullet + supervised forward dynamics + four-parameter [[two-arc-planar-motion-primitive]]) achieves per-cable median tip error of 22%/24%/34% of cable length across three cables (160 trials each). On the predecessor's planar (free-end-but-shorter-cable) Real2Sim2Real setting [[planar-robot-casting-real2sim2real-self-supervised]], the same pipeline family achieves 8%/12%/14% per-cable median tip error. Same Berkeley AUTOLAB lab, same overall recipe, same per-cable-length normalization metric. The free-end / longer-cable / PyBullet-only setting in this paper produces 2-3× higher median error."
  - source: "[[planar-robot-casting-real2sim2real-self-supervised]]"
    type: supports
    strength: moderate
    detail: "Provides the comparison baseline of 8-14% per-cable median tip error on the planar robot casting task (Lim et al., 2022). The predecessor uses Isaac Gym FleX-segmented (the strictly better simulator on the same hardware) and shorter cables, partially confounding the comparison."
conditions: "Comparison is between two papers from the same group (Berkeley AUTOLAB), evaluating on different cables, slightly different cable lengths (0.62-0.67 m vs. 0.63-0.65 m), different simulators (PyBullet only vs. PyBullet + Isaac Gym hybrid + Isaac Gym segmented; the predecessor selected the segmented simulator as best), and slightly different action parameterizations (4-parameter (θ_1, θ_2, r_2, ψ) vs. 5-parameter (θ_1, r_1, θ_2, r_2, α)). Both use the same per-cable-length normalization, same Real2Sim2Real pipeline pattern, similar UR5 hardware, similar reset procedure, similar overhead-camera tracking."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

Under the same Real2Sim2Real pipeline pattern, same per-cable-length error normalization, same Berkeley AUTOLAB lab, and similar UR5 hardware, **dynamic planar target reaching with a free-end cable produces 2-3× higher median tip error than the closely-related planar-robot-casting setting** that the same recipe targets, suggesting that fully-free-end dynamics (here, longer cables paired with the longer trajectories the longer cables permit) are intrinsically harder than the closer-to-rigid setting where the cable is shorter and the motion is single-arc-and-pull.

## Evidence summary

Two same-lab papers run substantially the same pipeline on the same problem family:

| Setting | Median tip error per cable (% cable length) | Source |
|---|---|---|
| PRC (free-end, ~0.65 m cables, 5-param action, Isaac Gym FleX-segmented) | 8% / 12% / 14% | [[planar-robot-casting-real2sim2real-self-supervised]] |
| Free-end planar (free-end, 0.62–0.67 m cables, 4-param action, PyBullet) | 22% / 24% / 34% | [[self-supervised-learning-dynamic-planar-manipulation]] |

The 2-3× gap is robust across all three per-paper cables and is not closed by mixed-data fine-tuning, repeatability selection, or coverage-maximizing system parameter tuning.

## Conditions and scope

The comparison is **not** a controlled ablation. Confounds include:

1. **Simulator difference**: the new paper uses PyBullet only; the predecessor benchmarked three simulators and *selected Isaac Gym FleX-segmented* as best (with PyBullet 9–13% behind on waypoint/endpoint metrics within the predecessor's own paper). Some of the 22–34% may be the simulator downgrade.
2. **Action parameterization difference**: 4-parameter $(\theta_1, \theta_2, r_2, \psi)$ vs. 5-parameter $(\theta_1, r_1, \theta_2, r_2, \alpha)$. The new paper drops $r_1$ as a free parameter (spline-implicit) and uses an absolute wrist target $\psi$ rather than a wrist offset $\alpha$.
3. **Cable difference**: similar lengths (0.62–0.67 m vs. 0.63–0.65 m) but different materials, masses, and endpoint geometry; in particular, Cable 3 in the new paper has a magnet endpoint that the simulator cannot model.
4. **Workspace and target difference**: 32 targets × 5 trials per cable in the new paper; 16 × 5 in the predecessor. Slightly different table dimensions (2.75 × 1.50 m vs. 2.45 × 1.55 m).

Therefore the claim is "free-end-as-currently-reported is harder", not "free-end is fundamentally harder". A controlled head-to-head with the predecessor's simulator and parameterization is needed to isolate the free-end-cable-specific component.

## Counter-evidence

- The predecessor itself targets *free-end* cables (the cable's distal end is unconstrained); both papers are technically free-end. The new paper's harder regime may be driven by **longer/heavier** cables (with the wire+magnet outlier), **PyBullet-only simulation**, and **longer trajectories** (~5 s vs. ~2 s motion + reset overhead) rather than free-end-ness per se.
- The fixed-end / fixed-endpoint case in [[robots-lost-arc-self-supervised-learning]] (Zhang et al. 2021) operates in a different normalization regime; that paper does not report median tip-to-target error in % of cable length, so the *fixed-end vs. free-end* comparison the title implies is not directly evaluable from these two ingest sources.
- A higher-fidelity simulator could plausibly close most of the gap: the predecessor showed Isaac Gym FleX-segmented gives 9–13% improvement over PyBullet at the same task, and a Cosserat-rod-based simulator (e.g. [[deformx-versatile-co-simulation-framework-deformable]]'s direction) is plausibly even better.

## Linked ideas

(none yet — a natural follow-up idea is a controlled ablation: same simulator, same parameterization, same hardware, swap only the cable / motion type)

## Open questions

- How much of the 2-3× gap remains after replacing PyBullet with Isaac Gym FleX-segmented or with [[differentiable-discrete-elastic-rods]]?
- How much remains after equalizing trajectory length and cable mass?
- Is there a true fixed-end vs. free-end comparison in the literature on the same hardware and pipeline? [[robots-lost-arc-self-supervised-learning]] is the natural anchor but uses a different metric.
- Does the same gap hold if both settings are evaluated under closed-loop visuomotor control?
