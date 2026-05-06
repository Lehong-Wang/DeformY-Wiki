---
title: "Hierarchical DLO Routing with Reinforcement Learning and In-Context Vision-language Models"
authors: "Mingen Li, Houjian Yu, Yixuan Huang, Youngjin Hong, Hantao Ye, Changhyun Choi"
venue: "arXiv (cs.RO); ICRA 2026 candidate"
year: 2025
arxiv_id: "2510.19268"
doi: null
note_type: bibliography_only
sources: [report-2, l1b-5]
---

# Hierarchical DLO Routing with Reinforcement Learning and In-Context Vision-language Models

**One-line gist**: Hierarchical autonomous DLO routing combining VLM in-context high-level planning into language sub-goals with RL low-level skills + failure-recovery sub-policy.

**Task setup**: Long-horizon DLO routing (cables/ropes through clip sequences) on real hardware. Vision provides observation; the goal is encoded as VLM-generated language sub-goals along the routing path.

**Sim vs real**: Both reported (sim training + real deployment per the project page).

**Learning method**: Hierarchical control. A vision-language model performs in-context high-level reasoning to synthesize feasible plans expressed as natural-language sub-goals; low-level RL skills execute each sub-goal; a failure-recovery mechanism reorients the DLO into insertion-feasible states.

**Action representation**: Low-level RL skills with end-effector motion primitives.

**Why cited in the surveys**: Important 2025 example of a different *target-conditioning* style for DLOs: instead of a 3D point or parametric shape, the target is encoded via VLM-grounded routing sub-goals. Cited as an example of how DLO target conditioning is broadening from pixel/keypoint to language + scene structure. Not a whip-target paper itself.

**Key result (if any)**: 92.5% overall success on long-horizon routing scenarios; ~50 percentage points over the next-best baseline.
