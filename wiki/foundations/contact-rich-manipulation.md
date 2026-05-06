---
title: "Contact-Rich Manipulation"
slug: "contact-rich-manipulation"
domain: "Robotics"
status: mainstream
aliases: ["compliant manipulation", "contact-rich tasks"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Contact-rich manipulation refers to tasks dominated by making and breaking contact — peg-in-hole, wiping, sliding, knot tying, suturing, deformable-object handling — where stiff position control fails and force/impedance/tactile feedback are essential.

## Intuition (LLM analysis)

Once contact matters, the dynamics become hybrid (smooth motion punctuated by impacts) and the policy must treat force as much as motion. Strategies range from impedance + force feedback to tactile-conditioned policies and contact-implicit trajectory optimization.

## Formal notation (LLM analysis)

Hybrid system: $\dot x = f_i(x, u)$ on mode $i$, with mode transitions governed by complementarity conditions $0 \le \lambda \perp \phi(x) \ge 0$ (signed distance and normal force).

## Key variants (LLM analysis)

- Hybrid trajectory optimization (DIRTREL, contact-implicit).
- Impedance- and force-feedback control.
- Tactile-conditioned policies.
- Diffusion / transformer policies on visuotactile inputs.
- Demonstration-driven / RL on contact-rich benchmarks (Robosuite, FurnitureBench).

## Known limitations (LLM analysis)

Simulators struggle with frictional contact fidelity. Reward shaping is fragile. Tactile sensors are still expensive and sparse. Long-horizon contact reasoning compounds.

## Open problems (LLM analysis)

Differentiable, faithful contact simulation; tactile foundation models; generalist policies that exploit force and slip cues; contact-rich whole-body humanoid manipulation.

## Relevance to active research (LLM analysis)

DLO manipulation — knotting, untangling, threading — is fundamentally contact-rich (rope-rope, rope-environment, rope-robot contacts). Advances here directly raise the ceiling for DLO research.
