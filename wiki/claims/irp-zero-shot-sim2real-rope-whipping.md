---
title: "Iterative Residual Policy enables zero-shot sim-to-real dynamic rope-tip targeting on UR5 across diverse rope properties and robot embodiments"
slug: "irp-zero-shot-sim2real-rope-whipping"
status: supported
confidence: 0.85
tags: [DLO, sim-to-real, dynamic-manipulation, rope-whipping, residual-learning, zero-shot, robot-embodiment]
domain: "Robotics"
source_papers: ["[[iterative-residual-policy-goal-conditioned-dynamic]]"]
evidence:
  - source: "[[iterative-residual-policy-goal-conditioned-dynamic]]"
    type: supports
    strength: strong
    detail: "On 7 unseen real-world ropes spanning material (cotton, leather, cloth), length (60-120 cm), mass distribution (uniform, knotted, bullwhip-tapered) and aerodynamics (long-cloth), IRP-9 reaches 1.3-5.5 cm mean tip-to-goal error after 9 iterations, vs. 8.4-22.5 cm for iterLinear-9 and 11.9-21.6 cm for OptSim (oracle exhaustive search in simulation with measured rope parameters). The same network — trained only on simulated ropes spanning length and density — generalizes zero-shot to all real ropes and to two un-trained robot embodiments (40 cm and 60 cm wooden extensions vs. the 50 cm trained extension), reaching 6.1 cm and 3.8 cm respectively at iteration 9. Headline numbers: 1.8 cm sim, 2.6 cm real-world. Recognized as RSS 2022 Best Paper and journal-extended in IJRR 2024."
conditions: "Parameterized 3-DoF whipping primitive (max angular velocity, joint 2 target angle, joint 3 target angle); UR5 robot with rigid stick extension; 2D trajectory targets in the Y-Z plane (out-of-plane goals reached via base joint rotation); fully observable rope tip via single RGB camera with 2D tracking; up to 10 real-world iterations per goal; goal precision target $\\leq$ 2 cm. Action repeatability is assumed (each action produces a similar trajectory each time)."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

A learned **delta dynamics network** combined with **iterative action refinement** and **adaptive action sampling**, all trained on a fixed simulator with linked-capsule rope physics over a 12×12 (length, density) parameter grid, transfers **zero-shot** to a real UR5 robot whipping diverse ropes — including ropes whose material, mass distribution, and aerodynamic properties are *out-of-distribution relative to training* — and to robot embodiments not seen at training time, achieving cm-level (≤6 cm) tip-to-goal accuracy within ≤9 iterations per goal.

## Evidence summary

The empirical evidence is unusually strong:

1. **Wide rope diversity.** 7 ropes including a leather bullwhip (non-uniform tapered mass distribution), a long-cloth (very low surface-area-to-mass ratio with strong aerodynamic drag, far OOD from any training rope), and a knotted base-rope (changing mass distribution mid-length).
2. **Wide goal range.** Targets across the workspace, including locations beyond the robot's reach (requiring the swing's dynamic momentum).
3. **Robot embodiment generalization.** Training on a 50 cm wooden extension; test on 40 cm and 60 cm extensions — these change *both* the action-to-trajectory mapping and the achievable trajectory envelope. IRP still converges (6.1 / 3.8 cm).
4. **Online recovery from system change.** Knotting the rope mid-experiment (step 6) changes its length, density, and mass distribution discontinuously; IRP regains accuracy within ~3 additional iterations, demonstrating genuine online adaptation rather than memorization.
5. **Outperforms an oracle.** OptSim performs exhaustive search in simulation using measured rope parameters and still loses to IRP, showing that the gap is in *un-modeled* physics (aerodynamics, floor collision) that no parameter-fitting approach can absorb but online trajectory observation can.
6. **Recognition.** RSS 2022 Best Paper Award, journal-extended in IJRR 2024 — independent peer-review validation that the result is reproducible and significant.

## Conditions and scope

The claim holds **only** under the listed conditions. Out of scope:

- **Closed-loop visuomotor control during the swing.** The action is a parameterized open-loop primitive; mid-swing feedback is not exploited.
- **3D action spaces.** The whipping primitive is 2D (Y-Z plane); 3D goals are reached only by base-joint rotation, not by 3D dynamics learning.
- **Non-repeatable tasks.** Stochastic outcomes, irreversible failures, or environment changes between iterations break the framework.
- **Cluttered or partially-observable scenes.** Trajectory rasterization assumes the rope tip is visible end-to-end.
- **Other deformables.** The cloth-placement task is also demonstrated, but the result is restricted to simulation and a smaller set of variations.

## Counter-evidence

None observed in the source paper. Plausible counter-stories the literature should test:

1. **Stronger linked-capsule baselines.** A heavily-tuned linked-capsule sysid baseline (different joint topology, learned damping, residual policy) might shrink the gap on a subset of ropes — but the OptSim oracle already represents the strongest version of that argument and IRP wins.
2. **Long-cloth or extreme aerodynamic regimes.** Performance on long-cloth (1.9 cm at iteration 9) is striking but a single test case; a broader aerodynamic-dominated study could falsify or strengthen the claim.
3. **Action repeatability assumption.** In environments with significant disturbance between iterations (wind, contact perturbations), iterations may not converge — not tested.

## Linked ideas

(none yet — the planned follow-up is the DeformY paper, which extends the rope-tip-targeting benchmark to closed-loop, full-3D control. The IRP iterative-refinement principle is a candidate building block for that line of work.)

## Open questions

- Does the same iterative-residual approach transfer to **closed-loop visuomotor policies** (per-timestep neural control)?
- Does the result hold for **3D action spaces** with non-planar dynamics, or does the trajectory rasterization break the formulation?
- Is the (length, density) training grid the *minimum sufficient* training distribution, or can it be much smaller?
- For the cloth task, can IRP achieve the same level of generalization to physical cloth properties (anisotropic stiffness, thickness, fiber type) as it does for rope?
- Does action repeatability hold for **contact-rich** dynamic deformable tasks (e.g., flicking, casting, tossing into a cluttered environment)?
