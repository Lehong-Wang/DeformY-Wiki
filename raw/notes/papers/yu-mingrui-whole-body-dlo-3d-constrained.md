---
title: "Generalizable Whole-Body Global Manipulation of Deformable Linear Objects by Dual-Arm Robot in 3-D Constrained Environments"
authors: "Mingrui Yu, Kangchen Lv, Changhao Wang, Yongpeng Jiang, Masayoshi Tomizuka, Xiang Li"
venue: "IJRR 2025"
year: 2025
arxiv_id: "2310.09899"
doi: null
note_type: bibliography_only
sources: [report-2]
---

# Generalizable Whole-Body Global Manipulation of Deformable Linear Objects by Dual-Arm Robot in 3-D Constrained Environments

**One-line gist**: Global planning + closed-loop manipulation pipeline for moving and shaping DLOs through 3D-constrained environments with a dual-arm robot; combines a simplified DLO energy model with an adaptive online motion model.

**Task setup**: Dual-arm robot moves and shapes a DLO in a 3D environment with obstacles/constraints. Goal = a target configuration consistent with the constraints.

**Sim vs real**: Real-robot validated; uses simulation for planning. Generalizes across various DLOs and constrained 3D scenes.

**Learning method**: Combines a simplified DLO energy model with an adaptive online DLO motion model — primarily model-based / classical, with adaptive components.

**Action representation**: Dual-arm joint / EE trajectories that achieve the target configuration through the constrained environment.

**Why cited in the surveys**: A recent flagship example of *configuration-conditioned* whole-body DLO manipulation in 3D, complementing tip-targeting and shape-control work. Demonstrates generalization across DLO types and 3D constraint scenes — relevant prior art for any 3D goal-conditioned DLO system.

**Key result (if any)**: Generalizable across various DLOs and constrained 3D scenes; real-robot validated.
