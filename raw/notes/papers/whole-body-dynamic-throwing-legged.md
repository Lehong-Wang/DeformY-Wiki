---
title: "Whole-Body Dynamic Throwing with Legged Manipulators"
authors: "ETH RSL group (lead-author overlap with the IROS 2025 ETH whole-body throwing paper)"
venue: "arXiv 2410.05681"
year: 2024
arxiv_id: "2410.05681"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Whole-Body Dynamic Throwing with Legged Manipulators

**One-line gist**: Sim-trained PPO RL policy for ANYmal-C + Kinova quadruped (and humanoid) throwing toward arbitrary 3D targets; whole-body coordination.

**Task setup**: Humanoid + ANYmal-C + Kinova quadruped throws an object to an arbitrary 3D target (up to 5 m radius); whole-body throw outperforms arm-only.

**Sim vs real**: IsaacGym sim-trained, sim2real to humanoid hardware; adaptive curriculum + sparse reward.

**Learning method**: PPO RL with adaptive curriculum + sparse reward; whole-body joint-velocity policy.

**Action representation**: Whole-body joint-velocity policy outputs.

**Why cited in the surveys**: Predecessor to the ETH IROS 2025 whole-body throwing paper. Cited as evidence that whole-body coordination beats arm-only on range/accuracy/stability for goal-conditioned throwing.

**Key result (if any)**: 73 cm error within 5 m radius; full-body throw outperforms arm-only.
