---
title: "MuJoCo 3 / MJX + cable composite plugin"
authors: "Google DeepMind / MuJoCo team"
venue: "Open-source software (Apache-2.0); v3.8.0 Apr 2026"
year: 2026
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, l2d-2]
---

# MuJoCo 3 / MJX cable composite plugin

**One-line gist**: MuJoCo's `cable` composite (plugin/elasticity/cable.xml) builds a chain of capsule segments connected by joints with bend and twist stiffness; MJX provides JAX/XLA differentiable variant on GPU.

**Task setup**: Simulator. Default cable plugin: 41-segment capsule chain, capsule radius 0.005, twist stiffness 1e7, bend stiffness 4e6, damping joint, slider body, equality constraint at one end.

**Sim vs real**: Sim only.

**Learning method**: None.

**Action representation**: N/A.

**Why cited in the surveys**: The most actively maintained "free" DLO simulator a roboticist can drop into an existing RL stack today (13.4k stars, v3.8.0 April 2026). MJX (mujoco_playground) gives GPU-batched JAX execution; MuJoCo Warp claims ~252×–475× speedups on Blackwell GPUs. As of MJX v0.2.0 (Mar 2026), no rope env ships first-class — but the cable plugin is usable from native MuJoCo (CPU) right now.

**Key result (if any)**: Standard cable composite + MJX-batched JAX execution. Code: https://github.com/google-deepmind/mujoco; https://github.com/google-deepmind/mujoco_playground.
