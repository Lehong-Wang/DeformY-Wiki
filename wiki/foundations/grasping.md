---
title: "Grasping"
slug: "grasping"
domain: "Robotics"
status: mainstream
aliases: ["robotic grasp", "grasp planning"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Grasp"
---

## Definition

A grasp is an act of taking, holding or seizing firmly with the hand. An example of a grasp is the handshake, wherein two people grasp one of each other's like hands.

## Intuition (LLM analysis)

A grasp is a set of contact points and forces that immobilizes the object against expected disturbances. Quality measures (force closure, form closure, Ferrari-Canny) quantify how well a grasp resists wrenches; learned grasp predictors generalize across object shapes.

## Formal notation (LLM analysis)

Force closure: $\exists \lambda_i \ge 0,\, \sum \lambda_i G_i = 0$ where $G_i$ are contact wrench bases. Quality $Q$ = radius of largest origin-centered ball inside the wrench convex hull.

## Key variants (LLM analysis)

- Analytic grasp planning (force/form closure).
- Learned grasp predictors (Dex-Net, GG-CNN).
- 6-DOF point-cloud grasping (Contact-GraspNet, AnyGrasp).
- In-hand manipulation / regrasping.
- Soft-finger and underactuated grasping.
- Grasping deformable objects (DLO, cloth) — pinching vs. wrapping vs. pinning.

## Known limitations (LLM analysis)

Sim-to-real gap for tactile/contact details. Grasping deformable / DLO targets cannot rely on rigid-object force closure. Dynamic re-grasping is largely unsolved.

## Open problems (LLM analysis)

Generalist 6-DOF grasping for cluttered scenes with deformables; tactile-conditioned regrasping; language-conditioned grasp selection.

## Relevance to active research (LLM analysis)

Most DLO manipulation tasks alternate between grasping the cable and reshaping it; grasp choice and regrasp planning materially affect downstream shape control.
