---
title: "Position-Based Dynamics"
slug: "position-based-dynamics"
domain: "Robotics"
status: mainstream
aliases: ["PBD", "XPBD"]
first_introduced: "2007"
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Soft-body_dynamics"
---

## Definition

Soft-body dynamics is a field of computer graphics that focuses on visually realistic physical simulations of the motion and properties of deformable objects. The applications are mostly in video games and films. Unlike in simulation of rigid bodies, the shape of soft bodies can change, meaning that the relative distance of two points on the object is not fixed. While the relative distances of points are not fixed, the body is expected to retain its shape to some degree. The scope of soft body dynamics is quite broad, including simulation of soft organic materials such as muscle, fat, hair and vegetation, as well as other deformable materials such as clothing and fabric. Generally, these methods only provide visually plausible emulations rather than accurate scientific/engineering simulations, though there is some crossover with scientific methods, particularly in the case of finite element simulations. Several physics engines currently provide software for soft-body simulation.

## Intuition (LLM analysis)

Time-step the positions naively, then iteratively snap them back to satisfy constraints (length, bending, volume, contact). The constraint solver replaces force-integration stiffness with geometric projections, yielding stable, real-time simulation suitable for games and robotics.

## Formal notation (LLM analysis)

Predict $\tilde x_i = x_i + \Delta t v_i + \Delta t^2 m_i^{-1} f_i^{ext}$. Solve constraints $C_j(\tilde x) = 0$ by iterative projection. Update $v_i = (\tilde x_i - x_i)/\Delta t$, $x_i \leftarrow \tilde x_i$. XPBD adds a compliance term that recovers physical stiffness.

## Key variants

- PBD (original Müller et al.).
- XPBD (extended, physically meaningful stiffness).
- GPU PBD (Flex, NVIDIA).
- Continuous-collision PBD.
- Differentiable PBD.

## Known limitations (LLM analysis)

Iteration count vs. effective stiffness trade-off. Constraint solver convergence is not guaranteed globally. Energy is not strictly conserved.

## Open problems (LLM analysis)

Differentiable PBD inside policy gradients for DLO control; PBD with rigorous physical fidelity for cables of arbitrary stiffness.

## Relevance to active research (LLM analysis)

Many real-time DLO simulators in robotics (Isaac Gym, MuJoCo MJX rope plugins, Genesis) lean on PBD/XPBD because of its stability under stiff cables and contacts.
