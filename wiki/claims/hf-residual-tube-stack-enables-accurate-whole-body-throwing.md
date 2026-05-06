---
title: "100 Hz nominal MPC + 400 Hz RL residual + pullback-tube optimizer enables 0.276 m mean landing error at 6 m on ANYmal-D + DynaArm"
slug: "hf-residual-tube-stack-enables-accurate-whole-body-throwing"
status: supported
confidence: 0.75
tags: [throwing, residual-policy, whole-body, pullback-tube, ANYmal, sim-to-real]
domain: "Robotics"
source_papers: ["[[learning-accurate-whole-body-throwing-high]]"]
evidence:
  - source: "[[learning-accurate-whole-body-throwing-high]]"
    type: supports
    strength: strong
    detail: "On hardware (ANYmal-D + Duatic DynaArm + Robotiq 2f140), targeting 4 m and 6 m ground locations with ±0.5 m lateral offset, 10 throws/target = 40 throws total. Mean landing error: 0.276 m at 6 m and 0.429 m at 4 m for the full proposed stack (100 Hz nominal RL policy + 400 Hz residual policy + closed-loop pullback tube acceleration optimizer). Nominal-only baseline: 0.685 m at 6 m and 0.710 m at 4 m. Average error reduction across distances: 49.5%. Additional 1500-condition stochastic simulation grid shows monotonic landing-error reduction with pullback control rate (96.8 cm at constant velocity → 31.1 cm at 400 Hz pullback). Human comparison: robot 71/125 (56.8%) vs 25 students 19/125 (15.2%) at 3-4 m floorball target, fixed 6 m/s release."
conditions: "Indoor venue with AprilTag-based target localization. Throwing speeds 6-10 m/s. 18-joint legged mobile manipulator (ANYmal-D + DynaArm + Robotiq 2f140 gripper). Training: legged_gym + PPO, 4500 nominal iterations + 1200 residual iterations. Standard sim-to-real techniques: actuator networks, domain randomization, observation noise, symmetry augmentation. Object inertia assumed small relative to EE inertia. Vertical-direction acceleration tracking only; horizontal handled by nominal."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

A three-layer control stack — (i) a 100 Hz nominal RL policy that tracks a time-dependent end-effector reference on the goal manifold of object flight, (ii) a 400 Hz residual policy trained on the frozen nominal that adds high-frequency arm-joint-position offsets, and (iii) a closed-loop pullback tube acceleration optimizer (sub-millisecond convex QP) that drives the EE state into the backward reachable tube during the 100 ms release window — enables **prehensile whole-body throwing with quantified accuracy** on a legged mobile manipulator. Concretely, on ANYmal-D + DynaArm + Robotiq 2f140 it achieves a **mean landing error of 0.276 m at a 6 m target**, a **49.5% accuracy improvement** over the nominal-only baseline, and a **3.7×** higher hit-rate than humans on a 3-4 m floorball target.

## Evidence summary

The hardware result spans 40 timed throws across 4 ground-target locations (4 m and 6 m forward distance with ±0.5 m lateral offset, 10 throws each). The full stack (nominal + residual + pullback) consistently outperforms the nominal-only baseline across both distances and across object types (gift boxes, snowballs, floorballs, indoor and outdoor).

The simulation grid (1500 conditions: 3 EE heights × 5 horizontal vels × 4 vertical vels × 25 perturbation ratios, 5 seeds, 2 m/s² Gaussian acceleration noise) shows a monotonic improvement of the pullback tube acceleration with control rate (constant-velocity baseline 96.8 cm → pullback at 400 Hz 31.1 cm in max landing error during the 50–100 ms detach window). The 500-trial simulated ablation at a 7 m target shows the full stack improves landing accuracy by 6.17% over the residual-alone variant and 20.04% over the nominal alone.

The human comparison (25 students, floorball, 3-4 m target, fixed 6 m/s release): robot 56.8% success vs humans 15.2%.

## Conditions and scope

The claim is verified under the conditions in `conditions` above. It is **not** yet shown that:

- the same stack transfers to substantially different legged manipulators (Spot + arm, humanoids, smaller quadrupeds);
- the result holds for non-projectile flight dynamics where the BRT cannot be expressed analytically;
- the residual policy's hardware vs simulation gap closes — currently the residual under-performs its simulation result on real hardware;
- the stack handles heavier or asymmetric objects (the formulation assumes object inertia ≪ EE inertia);
- the framework generalizes to the throwing of *deformable* objects (the snowball results suggest some robustness but with no quantified accuracy in the deformable regime).

## Counter-evidence

None directly observed in the source paper. Plausible counter-stories:

1. The 49.5% improvement is dominated by the pullback tube acceleration optimizer, with the high-frequency residual contributing only ~6.17% in the controlled ablation. A simpler stack (nominal + pullback, no residual) might suffice for the headline 0.276 m result, weakening the residual's claimed contribution.
2. The hardware throws are limited to indoor environment with AprilTag target localization; outdoor or vision-based target estimation may degrade accuracy meaningfully.
3. The residual policy's hardware gain at high velocity (16.8% velocity-tracking improvement at 10 m/s) is much smaller than its simulation gain — a residual-specific sim-to-real failure mode that the headline figure does not surface.
4. The human comparison was performed with partial implementations / earlier checkpoints due to timeline pressure, weakening the "robot beats humans" framing.

## Linked ideas

(none yet — this stack is a strong candidate template for whole-body dynamic deformable manipulation, but no concrete idea page has yet been opened.)

## Open questions

- Does removing the residual policy (and keeping only nominal + pullback) recover most of the headline accuracy, validating or undermining the residual's contribution?
- Does the architecture generalize to (a) cable / DLO whipping, (b) humanoid whole-body throwing, (c) outdoor visuomotor target estimation?
- Can the residual policy's sim-to-real gap be closed with simulator improvements (better actuator models, learned-dynamics correction) or with feedback-shaped residual structures?
- Is the 0.276 m residual landing error fundamentally limited by gripper variability + object inertia assumption, or by upstream tracking precision?
