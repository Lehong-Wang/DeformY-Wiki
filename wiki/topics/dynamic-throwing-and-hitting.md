---
title: "Dynamic Throwing and Hitting"
tags: [throwing, hitting, residual-physics, dynamic-manipulation, goal-conditioned, whole-body-control, motion-primitive, robot-learning]
key_venues: [RSS, IROS, ICRA, CoRL, T-RO]
related_topics:
  - "[[dynamic-dlo-tip-targeting]]"
  - "[[compact-action-parameterization]]"
  - "[[sim-to-real-and-rapid-adaptation]]"
key_people:
  - "[[andy-zeng]]"
  - "[[yuntao-ma]]"
  - "[[shuran-song]]"
key_papers:
  - "[[tossingbot-learning-throw-arbitrary-objects-residual]]"
  - "[[learning-accurate-whole-body-throwing-high]]"
  - "[[da-mmp-learning-coordinated-accurate-throwing]]"
  - "[[differentiable-motion-manifold-primitives-reactive-motion]]"
linked_ideas: []
---

## Overview

Goal-conditioned throwing and striking of rigid objects is the closest *methodological analog* to dynamic rope-tip targeting, and the source of the architectural template most likely to transfer: a low-rate analytic physics prior corrected by a high-rate learned residual. **[[tossingbot-learning-throw-arbitrary-objects-residual]]** (Zeng et al., RSS 2019 / T-RO 2020) is the canonical [[residual-physics]] paper — it predicts a scalar residual on the tangential release velocity over an analytic ballistic prior, reaching 84.7% real-world throw accuracy on seen objects. **[[learning-accurate-whole-body-throwing-high]]** (Ma et al., IROS 2025) generalizes that scalar residual into a continuous stack: a ~100 Hz nominal MPC plus a 400 Hz [[high-frequency-residual-policy]] and [[pullback-tube-acceleration]] for robust release on a legged manipulator (ANYmal-D + DynaArm), achieving 0.276 m mean landing error at 6 m. These works supply the dominant template — analytic prior + learned residual — that has not yet been transferred end-to-end to rope whipping, and they share the core challenge of committing to a precise dynamic action under release/impact uncertainty. They also motivate the **direction-conditioned** framing: a thrown or struck object's outcome is a pose, not just a position, which maps onto the position + direction target of the rope-tip-targeting direction.

## Timeline

| Year | Anchor result | What changed |
|---|---|---|
| 2019 | TossingBot | Residual physics over an analytic ballistic prior beats both pure-physics and pure-learning for goal-conditioned throwing |
| 2025 | ETH whole-body throwing | Scalar residual generalized to a continuous 100 Hz nominal MPC + 400 Hz residual + pullback-tube stack on a legged manipulator |

## Seminal works

- **[[tossingbot-learning-throw-arbitrary-objects-residual]]** (Zeng et al., RSS 2019 / T-RO 2020) — defines [[residual-physics]]: a learned residual on top of an analytic ballistic prior, jointly learned with grasping; 84.7% real-world throw accuracy.
- **[[da-mmp-learning-coordinated-accurate-throwing]]** (Chu & Xu, arXiv 2025) — first system to beat the residual-physics template on its own turf: **60.0% real ring-toss vs 6.7% for residual-style correction and 56.7% for trained human experts**, by generating whole trajectories from a learned manifold and conditioning on *executed* landings instead of correcting a commanded target.
- **[[differentiable-motion-manifold-primitives-reactive-motion]]** (Lee, ICRA 2026, DMMP) — 7-DoF Franka throwing to 1.1–2.0 m: 100 constraint-feasible joint trajectories in 0.012 s (+0.2 s verification) vs 10–3000 s for trajectory optimization; 100% success at 4 cm with rejection sampling. Simulation only, with the arm's end-effector velocity limits doubled.

## SOTA tracker

- **[[learning-accurate-whole-body-throwing-high]]** (Ma et al., IROS 2025) — 100 Hz nominal MPC + 400 Hz [[high-frequency-residual-policy]] + [[pullback-tube-acceleration]]; 0.276 m mean landing error at 6 m. The cleanest unexploited transfer template for whipping.

## Key benchmarks

- Real-world landing/throw accuracy (mean landing error at a target distance; success rate on seen vs novel objects) — the de-facto metric across the throwing line.

## Open problems

### Known gaps

- **Transfer to deformables**: the residual-physics and whole-body-throwing recipes are demonstrated on rigid objects; an end-to-end transfer to a deformable rope tip has not been published.
- **Pose/direction targets**: most throwing results target a landing *position*; conditioning on a full release/landing pose (position + direction) is less developed and directly relevant to the tip position + direction goal. **Partially addressed** by [[da-mmp-learning-coordinated-accurate-throwing]], which controls a 2-D landing position but explicitly leaves release orientation and spin uncontrolled and lists them as future work.

### Methodological gaps

- **Analytic prior for a deformable tip**: replacing TossingBot's ballistic velocity prior with a Cosserat-derived end-to-tip transfer at the snap instant is the cleanest unexploited transfer to rope whipping.
- **High-rate residual on a whip**: the ETH 100 Hz nominal + 400 Hz residual stack has not been attempted on a robot holding a whip.

## Concepts
- [[high-frequency-residual-policy]]
- [[residual-physics]]
- [[execution-outcome-conditioned-trajectory-generation]] — condition the generator on what the executed trajectory *actually achieved*, not on the target it was commanded toward.
- [[throwing-motion-primitive]]
- [[whole-body-prehensile-throwing]]
- [[pullback-tube-acceleration]]
