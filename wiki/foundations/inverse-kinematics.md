---
title: "Inverse Kinematics"
slug: "inverse-kinematics"
domain: "Robotics"
status: mainstream
aliases: ["IK"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Inverse_kinematics"
---

## Definition

In computer animation and robotics, inverse kinematics (IK) is the mathematical process of calculating the variable joint parameters needed to place the end of a kinematic chain, such as a robot manipulator or an animation rig's hand or foot, in a given position and orientation. IK operations are computationally much more complex than forward kinematics, in which joint parameters are trigonometrically calculated to achieve the position and orientation of the chain's end.

## Intuition (LLM analysis)

Forward kinematics is the easy direction: joints to pose. Inverse is hard because a pose may have many, one, or no joint solutions; redundancy lets us optimize secondary objectives (avoid limits, manipulability).

## Formal notation (LLM analysis)

Find $q$ such that $T_{ee}(q) = T^*$. For redundant manipulators, use $\dot q = J^+(q)\,\dot x + (I - J^+ J) z$, with $J^+$ a pseudoinverse and $z$ a null-space task.

## Key variants

- Analytical IK (closed-form for 6-DOF arms with spherical wrists).
- Damped least-squares / Levenberg-Marquardt.
- Optimization-based IK (TRAC-IK, BiT-RRT).
- Learned IK / neural-network priors.
- Whole-body / task-priority IK for humanoids and mobile manipulators.

## Known limitations (LLM analysis)

Singularities cause Jacobian rank loss. Redundancy resolution must be designed. Joint-limit, self-collision, and posture constraints turn IK into a constrained nonlinear program.

## Open problems (LLM analysis)

Globally consistent IK for under-actuated and continuum robots; differentiable IK for trajectory optimization on GPU; IK with deformable end-effectors.

## Relevance to active research (LLM analysis)

DLO manipulation typically issues end-effector targets that are then solved with IK. Choice of IK smoothness and null-space tasks materially affects how the rope is dragged through a configuration.
