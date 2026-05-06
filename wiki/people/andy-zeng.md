---
name: "Andy Zeng"
affiliation: "Google DeepMind (formerly Google AI / Princeton University)"
tags: [robotics, deep-learning, manipulation, vision, robot-learning, foundation-models]
homepage: "https://andyzeng.github.io/"
scholar: "https://scholar.google.com/citations?user=q7nFtUcAAAAJ"
date_updated: 2026-05-06
---

## Research areas

- Visuomotor manipulation policies (grasping, pushing, throwing, packing).
- Hybrid analytical–learned controllers and Residual Physics.
- Self-supervised robot learning at scale.
- Vision–language–action interfaces, code-generation for robotics, and foundation-model-driven robot policies.

## Key papers

- [[tossingbot-learning-throw-arbitrary-objects-residual]] — first author. RSS 2019 Best Systems Paper / IEEE T-RO 2020. Joint grasping+throwing with Residual Physics; 514 mean picks per hour.

## Recent work

(LLM analysis; not from this ingest's S2 fetch)

- Visual Pushing and Grasping (VPG) and Form2Fit on dense pixel-wise affordance prediction for manipulation.
- Implicit behavioral cloning, Transporter Networks, and CLIPort for category-level pick-and-place.
- Code-as-Policies and Socratic Models for LLM-driven robot programs and multimodal reasoning.

## Collaborators

- Shuran Song (Columbia) — long-running collaborator across grasping, pushing, throwing, packing.
- Johnny Lee (Google) — engineering and systems direction for Robotics at Google.
- Alberto Rodriguez (MIT, MCube Lab) — analytical manipulation mechanics and the Amazon Robotics Challenge line of work.
- Thomas Funkhouser (Google / Princeton) — Andy's PhD advisor; vision and robotics.

## My notes

Andy Zeng is the canonical reference point for the line of work that frames manipulation as dense pixel-wise affordance prediction shared across grasping, pushing, throwing, and packing — the architectural style TossingBot uses for joint grasp/throw learning. For DeformY, his residual-physics line is the primary methodological precedent; the *grasping-supervised-by-task-success* design pattern (TossingBot) is also directly applicable to DLO casting, where supervising base-motion choice by tip-target outcome could mirror the same effect.
