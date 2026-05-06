---
name: "Yuntao Ma"
affiliation: "Robotic Systems Lab, ETH Zürich"
tags: [legged-robotics, whole-body-control, residual-policy, sim-to-real, loco-manipulation]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

Whole-body loco-manipulation on legged mobile manipulators (ANYmal + DynaArm), reinforcement learning for dynamic manipulation, residual-policy stacks combining learning with model-based control, and dynamic skills such as throwing and badminton. Affiliated with ETH RSL (Marco Hutter's group).

## Key papers

- [[learning-accurate-whole-body-throwing-high]] — first author. Whole-body prehensile throwing with a 100 Hz nominal RL policy + 400 Hz residual policy + closed-loop pullback tube acceleration optimizer; 0.276 m mean landing error at 6 m on hardware. IROS 2025.

## Recent work

- IROS 2025: whole-body prehensile throwing with quantified hardware accuracy (this paper).
- Earlier ETH RSL work on legged-manipulator dynamic skills (referenced as `ma2025badminton` and `ma2022combining` in the throwing paper — episode-structuring conventions and decoupled loco-manipulation precedent).

## Collaborators

- Marco Hutter (ETH RSL, advisor on this paper)
- Kaixian Qu (ETH RSL)
- Yang Liu (EPFL)

## My notes

The throwing paper is a clean demonstration of the residual-on-frozen-nominal pattern at scale. Worth tracking for follow-on work on (i) closing the residual policy's sim-to-real gap, (ii) extending the pullback tube template to non-projectile or deformable-object flight dynamics, and (iii) generalizing the architectural template to other dynamic whole-body skills. The badminton precedent suggests the group is systematically pursuing dynamic loco-manipulation skills as a research arc.
