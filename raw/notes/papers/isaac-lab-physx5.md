---
title: "Isaac Lab / Isaac Sim / PhysX 5 (NVIDIA)"
authors: "NVIDIA Isaac team (Isaac Lab tech report arXiv 2511.04831)"
venue: "Open-source / arXiv 2025"
year: 2025
arxiv_id: "2511.04831"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Isaac Lab / Isaac Sim / PhysX 5

**One-line gist**: GPU-native, photoreal Isaac stack with FEM cloth, PBD ropes/inflatables, two-way coupling. Real2Sim2Real (Lim et al. 2022) used Isaac Gym for cable casting.

**Task setup**: General-purpose RL simulation framework.

**Sim vs real**: Sim only; widely used for sim2real transfer.

**Learning method**: None — sim/RL framework.

**Action representation**: N/A.

**Why cited in the surveys**: NVIDIA's standard high-fidelity, GPU-native robotics simulator. Cited as the substrate the Real2Sim2Real planar casting work used and as one of the leading options for GPU-batched DLO RL with photoreal rendering. Isaac Lab tech report arXiv 2511.04831 details current support.

**Key result (if any)**: GPU-native rigid + cloth + rope + soft-body simulation with two-way coupling.
