---
title: "Coordinated Manipulation of Hybrid Deformable-Rigid Objects in Constrained Environments"
authors: "Anees Peringal, Anup Teejo Mathew, Panagiotis Iiatsis, Federico Renda"
venue: "arXiv (cs.RO)"
year: 2026
arxiv_id: "2603.12940"
doi: null
note_type: bibliography_only
sources: [report-2, l2a-5]
---

# Coordinated Manipulation of Hybrid Deformable-Rigid Objects in Constrained Environments

**One-line gist**: Quasi-static optimization-based (NOT learning) planner for hybrid deformable-rigid linear-object assemblies via a strain-based Cosserat rod with analytical gradients; dual-arm hardware with a 3-link hDLO.

**Task setup**: Hybrid deformable-rigid linear-object assemblies (hDLOs — e.g., a rope segmented with rigid links). Dual-arm robot manipulates a 3-link hDLO toward a target configuration.

**Sim vs real**: Validated in sim across various hDLO systems and on real dual-arm hardware with a three-link hDLO.

**Learning method**: None — strain-based Cosserat rod model + analytical gradients for inverse kinetostatic / trajectory optimization.

**Action representation**: Joint-space dual-arm trajectory optimization with analytical gradients.

**Why cited in the surveys**: Not a learning paper, but the strain-based Cosserat model + analytical-gradient framework is a candidate differentiable simulator baseline for whip-to-target work. Quasi-static today, but the strain-Cosserat machinery is the same family used in MAT-DiSMech / DisMech and is gradient-friendly for direct policy / trajectory optimization.

**Key result (if any)**: ~3 cm avg deformation error (5% of deformable-link length); 33× speedup over finite-difference baselines.
