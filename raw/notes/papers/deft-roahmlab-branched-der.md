---
title: "DEFT: Differentiable Branched Discrete Elastic Rods for Modeling Furcated DLOs in Real-Time"
authors: "Yizhou Chen et al. (U-Mich ROAHM Lab)"
venue: "arXiv 2502.15037"
year: 2025
arxiv_id: "2502.15037"
doi: null
note_type: bibliography_only
sources: [report-2, l2d-4]
---

# DEFT — Differentiable Branched Discrete Elastic Rods

**One-line gist**: Extends DEFORM/DDER to branched DLOs (wiring harnesses); models junction-point dynamics and mid-rope grasping with PyTorch autograd.

**Task setup**: Branched DLO (BDLO / furcated cable) modeling and manipulation. Real-time autonomous wire-insertion demo.

**Sim vs real**: Real-time inference; planning demos on real branched cables.

**Learning method**: Physics-only, residual-physics, and full joint-learning training modes provided. Differentiable elastic-rod simulator (PyTorch autograd).

**Action representation**: Trajectory / policy-level — varies per usage.

**Why cited in the surveys**: Extends DEFORM to branched DLOs — important enabling technology for manipulation of wiring harnesses, beyond single-strand whip work. Cited as part of the DEFORM/DEFT family.

**Key result (if any)**: First public dataset and code for branched-DLO modeling, designed for real-time inference. Code: https://github.com/roahmlab/DEFT.
