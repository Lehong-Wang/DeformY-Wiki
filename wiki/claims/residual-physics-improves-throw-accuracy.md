---
title: "Learning a residual on top of an analytic ballistic prior yields high-accuracy goal-conditioned throwing of arbitrary objects"
slug: "residual-physics-improves-throw-accuracy"
status: supported
confidence: 0.85
tags: [robotics, throwing, residual-physics, manipulation, sim-to-real, hybrid-controller]
domain: "Robotics"
source_papers: ["[[tossingbot-learning-throw-arbitrary-objects-residual]]"]
evidence:
  - source: "[[tossingbot-learning-throw-arbitrary-objects-residual]]"
    type: supports
    strength: strong
    detail: "Real-world throwing accuracy: Residual-Physics 84.7% / 82.3% (seen / unseen objects) vs. Physics-only 61.3% / 58.5% vs. Regression-PoP 54.2% / 52.0%. Generalization to unseen target locations: Residual-Physics 87.2% / 83.9% (sim / real) vs. Regression-PoP 26.5% / 32.7%. Simulation ablations show Residual-Physics dominates on hammers (81.2%), rods (86.4%), unseen objects (66.5%), with the gap to baselines widening on hard, off-CoM objects. RSS 2019 Best Systems Paper, IEEE T-RO 2020 journal extension; replicated by independent labs (e.g. residual RL — Johannink et al., Silver et al.) on related task families."
conditions: "Holds for: rigid arbitrary objects in cluttered bin, RGB-D perception, fixed release height / distance / angle, target boxes within ballistic feasibility, UR5-class arm with parallel-jaw gripper, sufficient online self-supervised trial-and-error data (15k steps in simulation; comparable on a real robot). May not hold for: deformable / fragile objects, in-flight pose targeting, throwing primitives whose analytical prior is qualitatively wrong (e.g. high-drag projectiles where the linear-trajectory assumption fails), or extreme out-of-distribution target geometries."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

When a robot is trained end-to-end to grasp and throw arbitrary rigid objects from a cluttered bin into target boxes, augmenting an analytical ballistic controller with a learned, task-conditioned residual on the release velocity yields substantially higher throwing accuracy and substantially better generalization to unseen target locations than either the analytical controller alone, a from-scratch regression on release velocity, or a regression pre-trained on the analytical controller's outputs.

## Evidence summary

The TossingBot paper (RSS 2019 Best Systems / IEEE T-RO 2020) provides the primary evidence. Three independent test conditions all show Residual-Physics dominating:

1. **Real-world targeted throwing** (Table III). 84.7% / 82.3% throw success on seen / unseen objects (UR5 + RG2 gripper, 80+ real objects). Closest baseline (Physics-only) is 23–24 percentage points lower. Roughly matches or slightly exceeds an untrained 15-person human baseline.
2. **Generalization to unseen target locations** (Table IV). On novel boxes outside the training-time set, Residual-Physics is 87.2% (sim) / 83.9% (real) vs. Regression-PoP's 26.5% / 32.7%. This isolates the analytical prior's contribution — the residual alone cannot generalize this far without the prior.
3. **Simulation ablations across object difficulty** (Tables I and II). On the hardest seen object (hammers, off-CoM grasps): Residual-Physics 81.2%, Physics-only 70.4%, Regression-PoP 47.8%, Regression 32.8%. The gap widens with difficulty, showing the residual mainly compensates where the ballistic model is most wrong.

The system also achieves 514 mean picks per hour, more than 1.6× the previous SOTA grasp-and-place pipeline (Dex-Net 4.0, 312 MPPH), demonstrating the practical throughput consequence of accurate throwing.

Independent corroboration on related tasks: concurrent work on residual RL (Johannink et al., Silver et al., 2018) and follow-up Iterative Residual Policy (IRP, RSS 2022) extend the action-residual pattern to other dynamic manipulation tasks with similar gains.

## Conditions and scope

- **Object class**: rigid, throwable, robust to grasping/release forces.
- **Sensing**: RGB-D from a fixed-mount camera; calibrated landing tracker (overhead camera).
- **Hardware**: industrial-grade arm with millimetre repeatability; parallel-jaw gripper.
- **Action parameterization**: throw is reduced to a scalar release-velocity magnitude under fixed release height/distance/angle constraints. The residual is correspondingly low-dimensional.
- **Training regime**: online self-supervised trial-and-error with prioritized experience replay; ~15k training steps suffice in simulation.
- **Target distribution**: boxes within ballistic feasibility and not requiring obstacle avoidance in the projectile's path.
- **Excluded scope**: deformable / articulated / fragile objects; in-flight pose targeting; high-drag projectiles where linear-trajectory assumption qualitatively fails.

## Counter-evidence

No published *direct* counter-evidence as of this ingest. Soft caveats:

- The improvement over Physics-only is much smaller in pure simulation (lacking aerodynamics) than in the real world, so part of the residual's value comes specifically from compensating un-modelled real-world dynamics. Where simulators model dynamics well, the marginal benefit shrinks.
- Generalization to *qualitatively different* throwing modes (overhand, whip) is not tested, and the residual may not transfer.
- Beyond throwing, demonstrations of residual physics as a general control template are still narrow (block-assembly, peg-in-hole, planar pushing).

## Linked ideas

(none yet — to be filled in by `/ideate` follow-ups exploring residual-physics-on-DLO, e.g. residual base motion for cable casting)

## Open questions

- Does the analytical-prior + residual decomposition still dominate when the analytical prior is itself a *learned* low-fidelity simulator rather than closed-form physics?
- What is the minimum data budget for the residual to outperform pure regression as the analytical prior degrades from "exactly right" to "qualitatively wrong"?
- Can the residual be made *targeted-pose conditioned* (not just velocity-conditioned) to control landing orientation, without breaking the action's low-dimensional structure that gave the prior its generalization?
- Does the same evidence pattern (residual >> regression > physics-only on real objects) hold when the projectile is a *deformable* tip of a DLO rather than a rigid object?
