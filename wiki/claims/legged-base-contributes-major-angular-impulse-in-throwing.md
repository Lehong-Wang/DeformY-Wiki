---
title: "Whole-body coordination contributes ~53% additional angular impulse over arm-only throwing on a legged mobile manipulator"
slug: "legged-base-contributes-major-angular-impulse-in-throwing"
status: supported
confidence: 0.7
tags: [throwing, whole-body, base-motion, legged-manipulator, angular-impulse, loco-manipulation]
domain: "Robotics"
source_papers: ["[[learning-accurate-whole-body-throwing-high]]"]
evidence:
  - source: "[[learning-accurate-whole-body-throwing-high]]"
    type: supports
    strength: moderate
    detail: "Five hardware throws at 10 m/s horizontal velocity with ANYmal-D + DynaArm. Inverse dynamics computed via Pinocchio from joint positions, velocities, accelerations (finite differences with low-pass filter). The legged base contributes 51.7 N·m·s of angular impulse and 46.3 J of work to the arm prior to gripper release. Compared with a tabletop manipulator running the same joint command trajectory (counterfactual with zero base motion), the legged base provides 53.4% higher angular impulse. Qualitatively, the base stays largely stationary at low velocities and tilts as commanded velocity grows."
conditions: "Five throws at 10 m/s horizontal velocity. Single robot platform (ANYmal-D + DynaArm). Comparison performed against a counterfactual fixed-base run executing the identical arm joint trajectory, not against a separately optimized fixed-base policy. Onboard state estimation, joint accelerations from finite differences (noise-prone). Sample size n=5."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

For prehensile high-velocity throwing (≥10 m/s) on a legged mobile manipulator, **coordinated whole-body motion** (base lean / leg push / torso rotation in addition to arm motion) provides on the order of 50% more angular impulse to the end-effector than arm-only throwing of the same joint command trajectory. The legged base contributes approximately 51.7 N·m·s of angular momentum and 46.3 J of mechanical work during the pre-release phase. The contribution is regime-dependent: at low commanded velocities the base stays largely stationary and adds little; at high velocities the base meaningfully tilts and amplifies the throw.

## Evidence summary

The source paper performed inverse-dynamics analysis (Pinocchio) on five throws with horizontal velocity 10 m/s, computing torque, angular impulse, and torsional power on the arm-base interface from joint position/velocity/acceleration trajectories. Joint positions and velocities came from onboard state estimation; joint accelerations came from finite differences of velocity, low-pass filtered.

Numerical headline: 51.7 N·m·s angular impulse + 46.3 J work imparted by the base prior to release. Counterfactual: a tabletop manipulator executing the identical joint command trajectory was simulated; its angular impulse was 53.4% lower than the legged version. Qualitatively, the supplementary video shows the base posture barely changes at low speeds and tilts visibly as commanded throw velocity rises.

This is the most direct quantification to date of *what the legs are doing* in a learned whole-body throw.

## Conditions and scope

The claim is verified under the conditions in `conditions` above. It is **not** yet shown that:

- the contribution is comparable for *non-prehensile* whole-body throws (push, strike, whip);
- the percentage scales with payload mass (the experimental object was a floorball — light and rigid);
- the comparison generalizes from "same joint trajectory on tabletop" to "best fixed-base policy" — a separately optimized fixed-base controller might recover some of the gap by exploiting different release postures, narrowing the apparent advantage;
- the result transfers to other legged platforms (smaller quadrupeds, humanoids);
- the angular-impulse advantage corresponds to *accuracy* gains (the headline accuracy result depends on residual + pullback, not directly on whole-body angular impulse);
- the 5-throw sample size is sufficient to characterize variance — only one variance point is reported.

## Counter-evidence

None directly observed in the source paper. Plausible counter-stories:

1. A fixed-base manipulator with a more aggressive arm-side controller (faster joint accelerations, longer reach configurations) might recover most of the angular impulse without base motion. The "tabletop manipulator running the same joint command trajectory" baseline holds the *trajectory* fixed; a re-optimized arm trajectory could be competitive.
2. The 53.4% headline includes implicit base-arm dynamic coupling (base motion lengthens the effective lever arm); decomposing into "base contribution" vs "coupling-induced arm contribution" could shift the attribution.
3. With n=5 throws, the variance of the angular impulse is not characterized; the central estimate may shift with more samples.
4. The angular impulse is computed against a single throw direction; oblique or asymmetric throws may show different ratios.

## Linked ideas

(none yet)

## Open questions

- Does the same whole-body angular-impulse advantage hold for non-prehensile dynamic manipulation (whipping a flexible payload, kicking, hammering)?
- What is the cleanest decomposition of "base contribution" vs "base-induced arm coupling contribution" in inverse-dynamics analyses of loco-manipulation?
- How does the contribution scale with payload mass and platform mass? Is there a regime where the legged base hurts (when base oscillation dominates target accuracy)?
- Does an optimal fixed-base policy with rerun arm trajectory recover the full angular impulse, weakening the claim that whole-body is *required*?
