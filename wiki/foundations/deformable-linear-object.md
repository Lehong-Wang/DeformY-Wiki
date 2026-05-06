---
title: "Deformable Linear Object"
slug: "deformable-linear-object"
domain: "Robotics"
status: mainstream
aliases: ["DLO", "rope", "cable"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

A deformable linear object (DLO) is a slender deformable body whose configuration is dominated by a 1D centerline subject to bending and twisting — examples include rope, suture, surgical thread, wire harness, hose, and cable.

## Intuition (LLM analysis)

DLOs are infinite-dimensional in principle but admit compact representations (centerline curves, keypoints, latent codes). Their dynamics couple bending, torsion, and frictional contact, which makes them a benchmark for high-DOF manipulation.

## Formal notation (LLM analysis)

Centerline $\gamma:[0,L] \to \mathbb{R}^3$ with material frame $\{d_1, d_2, d_3\}$; bending and twist energies $E = \tfrac{1}{2}\int [EI \kappa^2 + GJ \tau^2]\,ds$. State is typically a discretized polyline plus material frames (Cosserat / Discrete Elastic Rods).

## Key variants (LLM analysis)

- Inextensible vs. extensible (rope vs. elastic cord).
- Plastic / hysteretic DLOs (wire that holds shape).
- Tangle / knot configurations (topological state).
- Multi-DLO (cable bundles, harnesses).

## Known limitations (LLM analysis)

Dynamics are stiff and contact-rich; perception is occluded by self-contact and tangling. Topological state (knots) is discrete and not differentiable.

## Open problems (LLM analysis)

Tractable representations that capture both shape and topology; data-driven DLO dynamics; cross-DLO generalization; manipulation policies that explicitly reason about knots and contacts.

## Relevance to active research (LLM analysis)

DLOs are the canonical deformable-manipulation testbed in robotics — knot tying, cable routing, wire harness assembly, suturing — and the focal class of this wiki's research direction.
