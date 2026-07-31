---
title: "Throwing Motion Primitive"
aliases: ["throwing primitive", "throw primitive", "release primitive", "parameterized throw motion", "projectile launch motion primitive"]
tags: [robotics, motion-primitive, throwing, dynamic-manipulation, manipulation, end-effector-trajectory]
maturity: active
key_papers: ["[[tossingbot-learning-throw-arbitrary-objects-residual]]", "[[da-mmp-learning-coordinated-accurate-throwing]]"]
first_introduced: "2019"
date_updated: 2026-07-30
related_concepts: []
parent_topic: "[[dynamic-throwing-and-hitting]]"
---

## Definition

A **throwing motion primitive** is a parameterized end-effector trajectory whose execution culminates in a planned **release position** $r$ and **release velocity** $v$, at which moment the gripper opens and an object is launched as a projectile. The primitive is parameterized by $\phi_t = (r, v) \in \mathbb{R}^6$ (or its constrained subset), and the primitive's execution layer is responsible for synthesizing a joint-space trajectory that achieves the desired $(r, v)$ at release time.

It is the throwing analog of the standard *grasping primitive* — a small, low-dimensional, learnable action that abstracts away the underlying motion-planning machinery from a higher-level perception / policy network.

## Intuition

End-to-end policies that output joint torques cannot easily learn dynamic, ballistic-style behaviors because (a) their action space is too high-dimensional, (b) the relevant performance signal (where the projectile lands) is delayed and sparse, and (c) the underlying physics is well-characterized. A throwing primitive collapses the action space to the small set of parameters that actually determine the projectile trajectory — typically just the release pose and twist — and lets the policy reason over those alone. The primitive's synthesis layer (e.g. an analytical reverse-curl trajectory + IK) handles the rest.

Geometric simplifications further reduce the dimensionality: in TossingBot, the release height $r_z$ and release-distance $\sqrt{r_x^2 + r_y^2}$ are fixed, the release-velocity direction is angled $45°$ upward in the throw plane, and the throw plane is aligned with the target — so the primitive is effectively parameterized by a single scalar $\|v_{x,y}\|$.

## Formal notation

Throwing primitive: $\phi_t = (r, v)$, with constraints

$$(r_{x,y} - p_{t_{x,y}}) \times v_{x,y} = 0, \quad \sqrt{r_x^2 + r_y^2} = c_d, \quad r_z = c_h, \quad \|v_{x,y}\| = v_z.$$

Under these constraints and assuming linear projectile motion under gravity acceleration $a$,

$$\theta = \arctan\!\left(\frac{p_y}{p_x}\right), \quad r_x = c_d \sin\theta, \quad r_y = c_d \cos\theta, \quad \|v\| = \sqrt{\frac{a(p_x^2 + p_y^2)}{r_z - p_z - \sqrt{p_x^2 + p_y^2}}}.$$

Execution layer: a curl–uncurl arm trajectory that brings the mid-point between the gripper fingertips to $r$ with velocity $v$ at the moment the gripper opens.

## Variants

- **Underhand swing throw** (TossingBot): inward curl, outward uncurl, gripper orthogonal to throw plane.
- **Overhand throw / pitch**: shoulder-driven, longer kinematic chain; richer release-velocity envelope but higher torque demand.
- **Whip / flick throw**: high acceleration burst on a single distal joint, used for low-mass / high-velocity projectiles or for free-end DLO casting.
- **Dart throw / pinpoint throw**: fixed release pose, small range of release velocities; the canonical "homogeneous-throw" primitive used in pre-TossingBot literature.
- **Whole-body throw**: legs + torso + arm contribute to release velocity (ETH whole-body throwing line of work). Couples balance control with throw planning.
- **Free-end DLO casting primitive**: the "object" is a flexible cable; the primitive parameters are the *base motion* whose dynamics propagate to the free end.

## Comparison

- **vs. grasping primitive**: both are parameterized motion abstractions, but throwing primitives are *dynamic* (release condition is a velocity, not a static contact configuration), so their parameterization includes velocity / momentum.
- **vs. closed-loop visuomotor control**: a throwing primitive is open-loop after planning; closed-loop visual servoing is impractical at projectile timescales.
- **vs. dynamic motion primitive (DMP)**: DMPs parameterize entire trajectories with attractor dynamics; throwing primitives focus on the *terminal* condition because the projectile decouples from the arm at release.

## When to use

- The task ends with a free-projectile phase whose outcome is determined almost entirely by the release condition.
- Closed-loop control during the projectile phase is impossible (no contact) or impractical.
- The robot's full joint trajectory does not need to be exposed to the policy; only the release condition matters.

Avoid when the task requires fine in-flight control of the projectile (no such control is possible), or when contact persists during release (e.g. catching, collaborative handovers).

## Known limitations

- **Open-loop after release.** No correction for in-flight perturbations (drag, gusts).
- **Geometric simplifications restrict throw envelope.** Fixed release height / distance / direction reduce the reachable set of landing locations.
- **Object-conditioned dynamics absorbed into the residual.** The primitive does not model how the object decouples from the gripper; that responsibility falls to the residual learner or to careful grasp selection.
- **Release timing.** Discrete gripper-open command; small timing errors map to large landing errors at long throws.

## Open problems

- **Pose-conditioned throws.** Most primitives only target landing position; controlling landing orientation is mostly open.
- **Bi-manual throwing primitives** for objects whose mass distribution or shape requires two hands.
- **Adaptive primitive parameterization** that lets the policy choose between underhand/overhand/whip primitives based on object class.
- **Throw primitives for DLO endpoints.** Defining a "throwing primitive" when the object is a deformable cable whose tip is the projectile.

## Key papers

- [[tossingbot-learning-throw-arbitrary-objects-residual]] — formalizes the underhand throwing primitive parameterized by release position and velocity, with geometric constraints reducing planning to a single scalar magnitude.

## My understanding

The throwing motion primitive is the action-space abstraction that lets Residual Physics work — without collapsing the action to release-condition parameters, the residual would be high-dimensional and would lose the cross-target generalization the analytical prior provides. For DLO casting and tip-targeting, the analogous abstraction is a *base motion primitive whose terminal condition predicts the rope tip's trajectory*; the right parameterization of this primitive is the open methodological question.
