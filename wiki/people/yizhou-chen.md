---
name: "Yizhou Chen"
affiliation: "Department of Robotics, University of Michigan, Ann Arbor (ROAHM Lab)"
tags: [robotics, deformable-linear-objects, differentiable-simulation, manipulation]
homepage: "https://roahmlab.github.io/DEFORM/"
scholar: ""
date_updated: 2026-05-06
---

## Research areas

- [[dynamic-deformable-object-simulation]]
- Deformable linear object (DLO) modeling and manipulation
- Differentiable physics for robotics — combining classical rod models with deep learning
- Real-time perception and closed-loop control of dynamic DLOs

## Key papers

- [[deform-differentiable-discrete-elastic-rods-real]] (CoRL 2024, first author)

## Recent work

DEFORM (CoRL 2024): the differentiable DER framework above, validated on five real DLOs with a Franka + Kinova dual-arm setup and the ARMOUR receding-horizon planner.

## Collaborators

- Ram Vasudevan (PI, ROAHM Lab, U-Michigan Robotics)
- Yiting Zhang, Zachary Brei, Tiancheng Zhang, Yuzhen Chen, Julie Wu — DEFORM co-authors

## My notes

Lead author of the methodologically most surgical DLO simulation paper in the CoRL 2024 batch I have ingested so far: identifies three precise failure modes of vanilla DER for dynamic 3-D rope simulation (no gradients, integration drift, momentum-violating inextensibility) and addresses each with a minimal, principled fix. The mass-ratio momentum-preserving PBD correction in particular is a one-line theoretical contribution that generalizes well beyond DLOs. ROAHM lab also produced ARMOUR (used for closed-loop control in DEFORM) — relevant if [[deformx-versatile-co-simulation-framework-deformable]] follow-on work needs a guaranteed-safe receding-horizon planner.
