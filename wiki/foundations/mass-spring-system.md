---
title: "Mass-Spring System"
slug: "mass-spring-system"
domain: "Robotics"
status: mainstream
aliases: ["particle-spring system", "lumped-mass model"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Soft-body_dynamics"
---

## Definition

Soft-body dynamics is a field of computer graphics that focuses on visually realistic physical simulations of the motion and properties of deformable objects. The applications are mostly in video games and films. Unlike in simulation of rigid bodies, the shape of soft bodies can change, meaning that the relative distance of two points on the object is not fixed. While the relative distances of points are not fixed, the body is expected to retain its shape to some degree. The scope of soft body dynamics is quite broad, including simulation of soft organic materials such as muscle, fat, hair and vegetation, as well as other deformable materials such as clothing and fabric. Generally, these methods only provide visually plausible emulations rather than accurate scientific/engineering simulations, though there is some crossover with scientific methods, particularly in the case of finite element simulations. Several physics engines currently provide software for soft-body simulation.

## Intuition (LLM analysis)

Each mass obeys Newton's laws under spring forces from its neighbors. Behavior is controlled by spring stiffness, damping, and topology of the connection graph. Crude but cheap and widely used.

## Formal notation (LLM analysis)

$m_i \ddot x_i = \sum_{j \in \mathcal{N}(i)} k_{ij}(\|x_j - x_i\| - \ell_{ij}^0)\,\hat{e}_{ij} - \gamma \dot x_i + f_i^{ext}$.

## Key variants

- 1D rope (chain of masses).
- 2D cloth (regular grid).
- 3D soft body (tetrahedral mesh of springs).
- Provot constraints (length-preserving correction).
- XPBD-style mass-spring (constraint solvers).

## Known limitations (LLM analysis)

Stiff springs require small time steps (numerical instability). Cannot model bending/torsion well without auxiliary springs. Behavior depends on mesh topology, not only material parameters.

## Open problems (LLM analysis)

Learned spring parameters from real-world rope behavior; mass-spring as a fast surrogate inside MPC for rope shaping.

## Relevance to active research (LLM analysis)

Despite its limitations, mass-spring is still the default DLO simulator inside many real-time robot control loops because it is trivial to differentiate and parallelize.
