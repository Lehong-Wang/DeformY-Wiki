---
name: "Ashish Kumar"
affiliation: "UC Berkeley (at time of this work)"
research_areas: [legged-locomotion, sim-to-real, robot-learning, reinforcement-learning, rapid-adaptation]
homepage: "https://ashish-kmr.github.io/"
scholar: ""
date_updated: 2026-06-16
type:
  kind: researcher
---

## Research areas

- [[sim-to-real-and-rapid-adaptation]]
Legged-robot locomotion and real-time sim-to-real adaptation. First author of RMA (Rapid Motor Adaptation), the seminal two-phase context-encoder method that distills a privileged base policy into a proprioception-history adaptation module for sub-second online adaptation on cheap quadruped hardware.

## Recent work

- [[rma-rapid-motor-adaptation-legged-robots]] — first author. RMA: privileged base policy + extrinsics encoder, then a 1-D CNN adaptation module regressing the extrinsics from a 0.5s proprioceptive history; zero-shot on the A1 across diverse real terrains. RSS 2021.

## My notes

Kumar is the originator of the RMA recipe that the deformable-object line ([[rapid-adaptation-particle-dynamics-generalized-deformable]], RAPiD) later specializes, and that the DeformY rope-targeting arc is the recommended route to repurpose (privileged rope embedding → calibration-inferred, then frozen).
