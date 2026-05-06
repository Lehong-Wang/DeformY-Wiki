---
title: "Reinforcement Learning to Adjust Parametrized Motor Primitives to New Situations"
authors: "Jens Kober, Andreas Wilhelm, Erhan Oztop, Jan Peters"
venue: "Autonomous Robots / IJCAI 2011"
year: 2012
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-2]
---

# Reinforcement Learning to Adjust Parametrized Motor Primitives to New Situations

**One-line gist**: Meta-parameter RL (cost-regularized kernel regression / reward-weighted regression) adjusts DMP global parameters (release angle + velocity) to new dartboard targets across multiple real robots.

**Task setup**: Dart throwing (and ball paddling) on real Barrett WAM, BioRob, CBi, and Kuka KR6 platforms, hitting different dartboard target positions.

**Sim vs real**: Real-robot only.

**Learning method**: Meta-parameter RL — cost-regularized kernel regression / reward-weighted regression — which adjusts DMP meta-parameters (release angle + release velocity) to new task contexts.

**Action representation**: DMP global meta-parameters: release angle and release velocity.

**Why cited in the surveys**: One of the canonical "meta-parameter RL on motor primitives for goal-conditioned throwing" references. Cited in Report 2 as foundational analog for compact meta-parameter learning (release-angle/velocity) — the same recipe IRP uses for swing parameters.

**Key result (if any)**: Reliably hits all dartboard target positions from 260 rollouts.
