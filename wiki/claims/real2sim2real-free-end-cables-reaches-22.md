---
title: "DE-tuned PyBullet + supervised forward dynamics yields 22-34% median tip error on free-end cable casting"
slug: "real2sim2real-free-end-cables-reaches-22"
status: supported
confidence: 0.7
tags: [DLO, free-end-cable, sim-to-real, real2sim2real, dynamic-manipulation, supervised-learning, robot-casting]
domain: "Robotics"
source_papers: ["[[self-supervised-learning-dynamic-planar-manipulation]]"]
evidence:
  - source: "[[self-supervised-learning-dynamic-planar-manipulation]]"
    type: supports
    strength: moderate
    detail: "Per-cable median tip-to-target error of 22% (Cable 1, 0.62 m polyester rope), 24% (Cable 2, 0.65 m polyester rope), and 34% (Cable 3, 0.67 m wire+magnet) of cable length on a UR5 with 32 targets × 5 trials per cable (160 trials each). Pipeline: PyBullet capsule-chain cable with 10 parameters tuned per cable via Differential Evolution against 60 real trajectories; 4-layer MLP forward dynamics model with 256 hidden units pretrained on |D_sim|=36,000 simulated trajectories then fine-tuned on |D_real|=200 real trajectories per cable; deployment grid-searches 50,000 candidate actions through f_forw and selects the predicted-closest. Beats analytic Polar Casting (43% median on Cable 1) by 21 pp and a Gaussian Process baseline on D_sim (29%) by 7 pp; the mixed-data MLP also has the lowest IQR and lowest tail (max 78% on Cable 1 vs. 159% for the GP). Action parameterization is the four-dimensional [[two-arc-planar-motion-primitive]] (θ_1, θ_2, r_2, ψ). Off-table trials (7 for Cable 2, 1 for Cable 3) are excluded from error analysis."
conditions: "Planar 2D workspace (2.75 m × 1.50 m foam-covered table), UR5 with single open-loop dynamic motion (~5 s motion + ~15 s reset per trial), four-parameter polar two-arc + wrist action, PyBullet capsule-chain cable simulator with DE-tuned 10-parameter dynamics, 4-layer MLP with 256 hidden units, |D_sim|=36000 + |D_real|=200 per cable, Hindsight-Experience-Replay-style relabeling (executed action becomes its own target), three specific cables (two polyester ropes with hooks, one braided iron wire with magnet endpoint). Targets restricted to r_d ≤ r_max + r_c (reachable). Reset procedure: lift, hang 3 s, polar cast, drag back."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

A pipeline of (i) DE-tuned PyBullet capsule-chain cable simulator, (ii) supervised forward-dynamics MLP trained on simulated data and fine-tuned on a small real-data set, and (iii) test-time grid-search over the four-parameter [[two-arc-planar-motion-primitive]], achieves **per-cable median tip-to-target error of 22% / 24% / 34% of cable length** on dynamic planar manipulation of free-end cables on a UR5, across three cables of differing material, mass, stiffness, and endpoint geometry.

## Evidence summary

The source paper runs 32 targets × 5 trials = 160 physical rollouts per cable on the same hardware and pipeline. Median errors of 22%, 24%, 34% of cable length are reported with first/third quartile bounds (Q1 9–24%, Q3 35–50%) and per-cable maxima of 78–89%. The mixed-data $f_{\rm forw}$ ($\dsim+\dreal$) cleanly outperforms (i) the analytic Polar Casting baseline (43% median on Cable 1, with hard reachability limits), (ii) the Gaussian Process forward model on $\dsim$ (29% median, but max 159%), and (iii) both single-source ablations ($f_{\rm forw}$ on $\dreal$ alone collapses to 59% median; on $\dsim$ alone to 29% with max 113%). Mixed-data is the only configuration with median below 30% **and** the smallest tail.

The DE step itself reduces median final $L^2$ endpoint error in simulation vs. real from PyBullet defaults of 25–35% to 12–15% per cable, providing the prerequisite that $\dsim$ is faithful enough for supervised pretraining to transfer.

## Conditions and scope

- The 22–34% range is the *median* per cable; per-target outliers reach 78–89% even after fine-tuning.
- Each cable requires its own DE tuning round, $\dreal$ collection (~200 trajectories ≈ 1+ hour), and $f_{\rm forw}$ fine-tuning. No demonstrated cross-cable generalization.
- Targets must be reachable: $r_d \le r_{\rm max} + r_c$.
- Off-table failures and occluded endpoints are excluded from error analysis (7 of 160 for Cable 2, 1 for Cable 3, plus uncounted occluded cases).
- Cable 3's magnet endpoint introduces unmodeled rotational stochasticity that explains its 34% median despite the smallest tuning gap; the claim's *upper* bound (34%) is mostly driven by this single endpoint geometry.
- The claim is restricted to PyBullet; the predecessor paper found Isaac Gym FleX-segmented strictly dominant on the same task family, but this is not re-evaluated here.

## Counter-evidence

- **Tail outliers** within the same paper: max errors of 78%/77%/89% per cable indicate that the pipeline does not eliminate cable-length-scale failures, only shifts the median.
- **Per-cable retraining** is required — no single trained $f_{\rm forw}$ generalizes across cables.
- **2-3× worse than the fixed-end / shorter-cable predecessor** [[planar-robot-casting-real2sim2real-self-supervised]]'s 8–14% on the same per-cable-length normalization. The same R2S2R recipe does not transfer the predecessor's error budget to the free-end setting (this is the basis for the related [[free-end-cable-target-reaching-harder]] claim).
- The claim does not extend to **3D** out-of-plane dynamics, **closed-loop** policies, or **non-rope** DLOs.

## Linked ideas

(none yet — natural follow-ups are (i) replacing PyBullet with a higher-fidelity simulator, (ii) closed-loop visuomotor or residual policies, (iii) cross-cable meta-learning to amortize per-cable retraining)

## Open questions

- Would Isaac Gym FleX-segmented (the predecessor's winner) cut the 22–34% error in half on the same task?
- Does the tail (78–89% max) collapse under closed-loop visuomotor control?
- Would replacing the hand-crafted [[two-arc-planar-motion-primitive]] with a learned primitive (small VAE / DMP) at the same data budget help, hurt, or wash?
- How much of the gap from the predecessor's 8–14% is attributable to the free-end setting itself vs. the simulator downgrade vs. the longer/heavier cables?
