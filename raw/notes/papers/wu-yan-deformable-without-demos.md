---
title: "Learning to Manipulate Deformable Objects without Demonstrations"
authors: "Yilin Wu, Wilson Yan, Thanard Kurutach, Lerrel Pinto, Pieter Abbeel"
venue: "RSS 2020"
year: 2020
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Learning to Manipulate Deformable Objects without Demonstrations

**One-line gist**: Iterative pick-place RL action space for quasi-static deformable manipulation (rope, cloth) without human demonstrations.

**Task setup**: Quasi-static deformable manipulation tasks (rope smoothing, knot tying, cloth folding) using an iterative pick-place primitive.

**Sim vs real**: Sim-trained; real-world deployment limited to pick-place primitive transfer.

**Learning method**: Reinforcement learning over an iterative pick-place action space without demonstrations. Goal-image conditioning.

**Action representation**: Iterative pick-place — at each step, choose a pick point and a place point.

**Why cited in the surveys**: Cited in Report 1 as a methodologically related quasi-static deformable manipulation baseline. The iterative pick-place action space is a recurring design choice across early DLO RL work.

**Key result (if any)**: Demonstrates demonstration-free RL training on quasi-static rope/cloth manipulation tasks.
