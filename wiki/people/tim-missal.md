---
name: "Tim Missal"
affiliation: "TU Darmstadt (student); part of work performed at UNICAMP exchange"
tags: [DLO, world-model, RSSM, latent-dynamics, robot-learning]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

- Latent dynamics models for deformable linear objects
- Recurrent State Space Models (Dreamer family) for robotics
- Quaternionic kinematic-chain representations for shape-preserving dynamics

## Key papers

- [[ropedreamer-kinematic-recurrent-state-space-model]] — co-first author (with Lucas Domingues). Proposes a Quaternionic Kinematic RSSM with a dual-decoder architecture for long-horizon DLO dynamics; reports 40.52% RMSE reduction at 50 steps and 31.17% inference-time reduction over GA-Net/IN-BiLSTM baselines on a MuJoCo benchmark.

## Recent work

The RopeDreamer preprint (arXiv 2604.28161, April 2026) is the only paper currently associated with this author in the wiki's evidence base. The work was performed jointly between TU Darmstadt and UNICAMP (Brazil) as part of an exchange, with Honda Research Institute Europe involvement.

## Collaborators

- Lucas Domingues (UNICAMP / Instituto de Pesquisas Eldorado) — co-first author on RopeDreamer
- Berk Guler (TU Darmstadt / Honda Research Institute Europe)
- Simon Manschitz (Honda Research Institute Europe) — research scientist with prior work on imitation learning and skill transfer
- [[jan-peters]] (TU Darmstadt; DFKI; hessian.AI; Robotics Institute Germany) — senior author / supervisor
- Paula Dornhofer Paro Costa (UNICAMP) — co-corresponding senior author

## My notes

Lead student author of [[ropedreamer-kinematic-recurrent-state-space-model]]. The contribution that stands out for the DeformY arc is the architectural ablation (GA-Net + quaternionic representation collapses to 0% topology accuracy by $t=15$), which cleanly attributes long-horizon stability to the RSSM latent rather than the representation alone — a result that is more useful for downstream design choices than the headline RMSE number.
