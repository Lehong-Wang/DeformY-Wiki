---
title: "Jacobian-Based Control"
slug: "jacobian-based-control"
domain: "Robotics"
status: mainstream
aliases: ["resolved-rate motion control", "task-space control"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Jacobian-based control is a family of velocity / force control laws that map task-space objectives (end-effector velocity, force, or shape error) to joint-space commands using the manipulator Jacobian.

## Intuition (LLM analysis)

The Jacobian linearizes the kinematic chain at the current configuration. Small task-space changes translate to joint-space changes via $J$ (or $J^{-1}$ / $J^+$). Damping handles singularities; weighting trades off conflicting tasks.

## Formal notation (LLM analysis)

Velocity: $\dot x = J(q)\,\dot q$, so $\dot q = J^+\,\dot x$. Force: static $\tau = J^\top F$. Damped least squares: $\dot q = (J^\top J + \lambda^2 I)^{-1} J^\top \dot x$.

## Key variants (LLM analysis)

- Resolved-rate motion control (Whitney 1969).
- Damped-least-squares / pseudo-inverse control.
- Task-priority / null-space projection.
- Whole-body operational-space control.
- Deformation Jacobian (shape-servoing): map robot motion to deformable-object shape change.

## Known limitations (LLM analysis)

Requires accurate Jacobian. Singular near kinematic boundaries and when tasks conflict. For deformables, the deformation Jacobian must be estimated on-line and is sensitive to noise.

## Open problems (LLM analysis)

Online learning of deformation Jacobians for DLOs; safe Jacobian control under contact; combining Jacobian control with learned residuals.

## Relevance to active research (LLM analysis)

Shape-servoing of DLOs is fundamentally Jacobian-based control with a learned or estimated deformation Jacobian replacing the analytical kinematic Jacobian.
