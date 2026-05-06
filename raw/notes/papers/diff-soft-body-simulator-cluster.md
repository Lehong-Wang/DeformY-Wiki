---
title: "Differentiable Soft-Body / Cloth Simulator Cluster — DiffCloth, PlasticineLab, DiffPD, SoftMAC"
authors: "Multiple — DiffCloth (Li et al.), PlasticineLab (Huang et al.), DiffPD (Du et al.), SoftMAC"
venue: "Various — ICLR/CoRL 2020–2024"
year: 2022
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Differentiable Soft-Body / Cloth Simulator Cluster

**One-line gist**: Cluster of differentiable soft-body / cloth simulators including DiffCloth, PlasticineLab, DiffPD, and SoftMAC; soft-body-leaning rather than DLO-leaning.

**Task setup**: Each provides differentiable simulation of soft bodies (cloth, plasticine, projection-dynamics solids, multi-physics aggregations) for trajectory optimization or RL.

**Sim vs real**: Sim only.

**Learning method**: Generally support differentiable physics gradient flow into policy training.

**Action representation**: N/A.

**Why cited in the surveys**: Listed in Report 1 as the soft-body / cloth-leaning differentiable simulator constellation alongside DaXBench. Methodologically related to rope simulators but generally not optimized for the high-curvature, high-speed rope regime needed for whipping.

**Key result (if any)**: Differentiable soft-body / cloth dynamics across diverse representations.
