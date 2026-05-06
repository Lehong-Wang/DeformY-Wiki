---
title: "Two-stage self-curriculum MBRL + Jacobian visual servoing achieves zero-shot sim-to-real DLO shape control across varied size and material"
slug: "two-stage-mbrl-jacobian-servo-zero-shot-dlo-shape-control"
status: proposed
confidence: 0.45
tags: [DLO, MBRL, sim-to-real, shape-control, curriculum-learning, visual-servoing, robot-learning]
domain: "Robotics"
source_papers: ["[[self-curriculum-model-based-reinforcement-learning]]"]
evidence:
  - source: "[[self-curriculum-model-based-reinforcement-learning]]"
    type: supports
    strength: moderate
    detail: "30/30 real-world zero-shot success across three different DLOs — electric wire (purely elastic), USB cable, braided cotton rope (mild plasticity) — with 5 straight-init + 5 diverse-init cases per DLO and half of the diverse cases involving opposite curvatures. Avg min-error ≈ 2 mm under both initial conditions vs. 13–61 mm for MPC, Visual-Servo, and RL-Only baselines. Policy trained purely in MuJoCo with chained-capsule rope (40 capsules, 0.5 m length, 10 mm dia, bending stiffness 5×10^6) — no real-world data, no fine-tuning."
conditions: "Dual-arm planar (2D) shape control with two UR5/UR5e robots; 13 keypoints tracked by overhead RealSense D415 + color segmentation; primarily-elastic DLOs (does not extend to extremely low-stiffness DLOs); end-effectors confined to disjoint safe boxes preventing collision and overstretching; 250-step budget; success threshold 10 mm RMSE; stage-transition threshold 30 mm RMSE; small-deformation Jacobian gain λ=0.05; visual servo Jacobian warm-started during the RL stage."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

A **two-stage controller** — a model-based RL policy with self-curriculum goal generation handling large deformations, handing off to a Jacobian-based visual servo at small deformations — produces DLO shape-control policies trained purely in simulation that **transfer zero-shot to real hardware with 100% success rate (30/30) and millimetre-class accuracy**, across DLOs that vary in size, material, and degree of plasticity. The result holds for both straight-line and diverse initial shapes, and explicitly covers the difficult opposite-curvature regime that has historically defeated single-stage shape-control methods.

## Evidence summary

The source paper trains a single policy in MuJoCo with a chained-capsule rope and deploys it without retraining or fine-tuning to three real DLOs:

| DLO | Type | Avg min err — straight init | Avg min err — diverse init | Success — straight | Success — diverse |
|---|---|---|---|---|---|
| Electric wire | purely elastic | ~2 mm | ~2 mm | 5/5 | 5/5 |
| USB cable | mild plasticity | (combined into per-init averages) | (same) | 5/5 | 5/5 |
| Braided cotton rope | mild plasticity | (combined) | (same) | 5/5 | 5/5 |

Aggregated per init condition (across all three DLOs, n=15 each):

- Straight init: avg min err 1.99 mm, **15/15 success**
- Diverse init: avg min err 2.06 mm, **15/15 success**

Comparators on the same 30-case real-world test (each n=15 per init):

- MPC: 12/15 + 8/15 = 20/30 success, ~13–23 mm avg error
- Visual Servo (single-stage): 4/15 + 2/15 = 6/30 success, ~39–61 mm avg error
- RL-Only (single-stage): 6/15 + 3/15 = 9/30 success, ~16–38 mm avg error

The two-stage method outperforms every single-stage baseline by 10× on success count and ≥6× on mean error. The training pipeline (MBPO + Bi-LSTM ensemble + self-curriculum), the simulator (MuJoCo capsule chain), the calibration assumptions, and the small-deformation Jacobian-servo controller are all held constant across DLOs.

This is the strongest evidence to date that **a tightly task-decomposed two-stage controller can absorb the sim-to-real gap of dynamic dual-arm DLO shape control** with no domain randomization, no real-world demonstrations, and no online adaptation beyond the visual-servo Jacobian update.

## Conditions and scope

The claim applies under the conditions in `conditions` above. It is **not** yet shown that the same advantage holds for:

- 3D shape control (the entire benchmark is planar);
- highly plastic DLOs (cotton rope is the upper limit tested; clay-like cables are out of scope);
- non-elastic regimes generally;
- DLOs of substantially different aspect ratio (very short ropes, very long cables);
- non-MuJoCo simulators or non-capsule-chain rope models;
- closed-loop dynamic manipulation tasks with explicit time constraints (the success metric here is "reach within 250 steps", not "track a moving target");
- workspaces where the gripper-distance constraint cannot be enforced;
- vision pipelines other than overhead RealSense D415 with color segmentation.

The strongest single conceptual restriction is the **2D planar assumption** — extending to 3D involves substantially harder dynamics modeling and a richer action space, and the source paper explicitly flags this as future work.

## Counter-evidence

None directly observed in the source paper. The most plausible counter-stories:

1. **The visual-servo handoff is doing most of the work.** If the RL policy can reach $e < 30$ mm coarsely, the Jacobian servo's local linearization will close to mm-level on its own. An ablation running RL-Only at the same 10-mm success threshold (this paper does report this and it gets 9/30) shows the RL stage *cannot* match the two-stage performance, which is consistent with the framing — but a sharper test would be running the visual-servo handoff with a *much weaker* RL stage to see how robust the handoff really is.
2. **MuJoCo bias.** All training is in MuJoCo, all real-world deployments use the same vision and robot stack. If the rope's chained-capsule MuJoCo model accidentally happens to match the test DLOs' bending dynamics, the result may not generalize to other simulators or to DLOs whose dynamics are poorly captured by chained capsules (especially long ropes, where Cosserat-style continuum effects matter — the [[deformx-versatile-co-simulation-framework-deformable]] paper argues exactly this for *dynamic* DLO control).
3. **Planar-only confound.** The 2D restriction removes torsion, twist, and out-of-plane buckling, which are precisely the regimes where chained-capsule simulators are weakest. Whether 30/30 holds at 3D is an open question.
4. **No real-world variability test.** All three DLOs are tested in the same lighting/calibration; sensitivity to vision-tracking error, gripper slip, and dynamic perturbations is not characterized.

## Linked ideas

(none yet — the planned follow-up is the DeformY paper, which would combine this paper's MBRL recipe with the Cosserat-Isaac substrate to extend to dynamic 3D DLO shape control.)

## Open questions

- Does the two-stage decomposition still recover 30/30 success in 3D, where bending–torsion coupling, gravity, and out-of-plane dynamics break the planar simplifications?
- How robust is the RL → visual-servo handoff if the RL stage reaches the threshold *unstably* (e.g., the rope is in a transient overshoot when $e$ first crosses $\epsilon$)?
- What is the smallest set of real-world demonstrations that, combined with this MBRL curriculum, would let one *avoid* the visual-servo handoff entirely (i.e., make the RL stage itself sub-mm-accurate)?
- Does this advantage survive transfer between simulators (MuJoCo capsule → Cosserat rod → FEM)? If yes, it suggests the result is genuinely about the two-stage architecture; if no, it is partly an artifact of MuJoCo–real coincidence.
- Could the visual-servo stage be replaced by a *learned* small-deformation controller (e.g. a fine-tuned diffusion policy) without losing the zero-shot transfer?
