---
title: "Domain Randomization"
slug: "domain-randomization"
domain: "Robotics"
status: mainstream
aliases: ["DR"]
first_introduced: "2017"
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Domain_adaptation"
---

## Definition

Domain adaptation is a field associated with machine learning and transfer learning. It addresses the challenge of training a model on one data distribution and applying it to a related but different data distribution.

## Intuition (LLM analysis)

Instead of trying to match simulation to reality, exaggerate variability in simulation: lights, textures, masses, friction, latency. A policy that survives this distribution treats the real world as just another sample.

## Formal notation (LLM analysis)

$\theta^* = \arg\max_\theta \mathbb{E}_{\xi \sim p(\xi)} \mathbb{E}_{\tau \sim \pi_\theta, P_\xi} R(\tau)$.

## Key variants

- Visual DR (textures, lighting, camera).
- Dynamics DR (mass, friction, damping, latency).
- Adaptive / curriculum DR (gradually expand the distribution).
- Privileged-teacher DR.
- Active DR / bilevel learning of $p(\xi)$.

## Known limitations (LLM analysis)

Excessive randomization yields conservative policies. Defining the right axes to randomize is itself research. DR cannot fix a simulator missing the relevant physics altogether (e.g. cable elasticity).

## Open problems (LLM analysis)

Auto-tuning $p(\xi)$ from a small amount of real data; DR for contact-rich and deformable scenarios without overshooting computational budget.

## Relevance to active research (LLM analysis)

DR is a default tool for transferring rigid-body manipulation policies; for DLO manipulation, DR over rope length, stiffness, friction, and vision is an active research frontier.
