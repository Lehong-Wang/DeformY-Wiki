---
title: "Elastica / PyElastica — Open-Source Cosserat Rod Simulator"
authors: "Naughton, Sun, Tekinalp, et al. (Gazzola Lab, UIUC)"
venue: "IEEE RA-L 2021"
year: 2021
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, report-2, l2d-3]
---

# Elastica / PyElastica — Open-Source Cosserat Rod Simulator

**One-line gist**: Open-source Python implementation of Cosserat rod theory; widely used for soft robots, octopus-arm control, magnetic soft robots, and soft-robot RL benchmarks.

**Task setup**: Simulator (not a single task). Models assemblies of slender 1D bodies including external/internal forces and self-contact.

**Sim vs real**: Sim only; production-grade Python codebase.

**Learning method**: None — simulator. Companion `gym-softrobot` provides Gymnasium environments for soft-robot RL.

**Action representation**: N/A (simulator).

**Why cited in the surveys**: Cited as the most production-grade open-source Cosserat-rod simulator (335 stars, active 2025 development). Direct usability for rope-whipping policies if extended with a rigid robot arm.

**Key result (if any)**: Stable Cosserat-rod dynamics with self-contact. Code: https://github.com/GazzolaLab/PyElastica (v0.3.3, Aug 2025).
