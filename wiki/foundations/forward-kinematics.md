---
title: "Forward Kinematics"
slug: "forward-kinematics"
domain: "Robotics"
status: mainstream
aliases: ["FK", "direct kinematics"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Forward_kinematics"
---

## Definition

In robot kinematics, forward kinematics refers to the use of the kinematic equations of a robot to compute the position of the end-effector from specified values for the joint parameters.

## Intuition (LLM analysis)

Each joint applies a rigid transform to its successor; chaining them — typically via Denavit-Hartenberg or product-of-exponentials — gives the end-effector pose in closed form.

## Formal notation (LLM analysis)

$T_{ee}^0(q) = \prod_{i=1}^{n} A_i(q_i)$ where $A_i \in SE(3)$ is the joint-$i$ transform.

## Key variants

- Denavit-Hartenberg parameterization.
- Product of exponentials (Murray-Li-Sastry).
- Spatial vs. body Jacobian.
- URDF / kinematic tree libraries (Pinocchio, KDL, MuJoCo MJX).

## Known limitations (LLM analysis)

Requires accurate calibration of link lengths, joint offsets, and zero positions. Encoder noise propagates. Closed-chain mechanisms (parallel robots) have implicit FK constraints.

## Open problems (LLM analysis)

Differentiable, batched FK on GPU for large policy training; learned residual kinematic models for compliant / cable-driven robots.

## Relevance to active research (LLM analysis)

Foundational for any DLO manipulation system — the Jacobian and FK underpin shape-servoing control laws and provide the proprioceptive input fed to learned policies.
