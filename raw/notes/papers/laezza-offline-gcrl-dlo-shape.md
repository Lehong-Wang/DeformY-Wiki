---
title: "Offline Goal-Conditioned Reinforcement Learning for Shape Control of Deformable Linear Objects"
authors: "Rita Laezza, Mohammadreza Shetab-Bushehri, Gabriel Arslan Waltersson, Erol Özgür, Youcef Mezouar, Yiannis Karayiannidis"
venue: "arXiv (cs.RO)"
year: 2024
arxiv_id: "2403.10290"
doi: null
note_type: bibliography_only
sources: [report-1, report-2, l1b-2]
---

# Offline Goal-Conditioned Reinforcement Learning for Shape Control of Deformable Linear Objects

**One-line gist**: Frames planar DLO shape control as goal-conditioned offline RL with TD3+BC and HER-style data augmentation; explicitly handles curvature-inversion targets that defeat Jacobian shape-servoing.

**Task setup**: Planar shape control of two physical DLOs (a soft rope and an elastic cord) where the goal is a target shape encoded as a discretized planar polyline / keypoint set. Real robot deployment is part of the pipeline.

**Sim vs real**: Hybrid — real-robot data collection augmented with HER-style relabeling for offline training. Validated on real DLOs.

**Learning method**: Offline RL using TD3+BC, with HER-style data augmentation to limit real-robot data needs. The policy is conditioned on the target shape.

**Action representation**: Quasi-static end-effector pose / motion of the manipulating arm. Observation is the tracked planar DLO state.

**Why cited in the surveys**: Canonical example of *shape-conditioned* (rather than tip-point-conditioned) goal-conditioned RL for DLOs. Demonstrates HER-style relabeling on real-robot data — a recipe that recurs across the goal-conditioned DLO literature. Quasi-static and complementary to the dynamic-whip line.

**Key result (if any)**: Outperforms shape-servoing baseline on a curvature-inversion experiment where standard Jacobian-based methods fail.
