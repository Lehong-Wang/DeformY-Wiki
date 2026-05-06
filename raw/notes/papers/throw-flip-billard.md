---
title: "Learning to Throw-Flip"
authors: "Yang Liu, Bruno Da Costa, Aude Billard"
venue: "arXiv (likely IROS/ICRA 2026)"
year: 2025
arxiv_id: "2510.10357"
doi: null
note_type: bibliography_only
sources: [report-2, l1c-4]
---

# Learning to Throw-Flip

**One-line gist**: Real arm throws an object to a target 6-DoF pose using impulse-momentum motion design that decouples linear and angular release velocity, with regression learning a residual on the free-flight model.

**Task setup**: A real revolute robot arm throws an object so it lands at a desired *6-DoF pose* (3D position + orientation). Object in-hand spin is exploited; the system addresses the parasitic rotation that normally entangles release velocity with spin.

**Sim vs real**: Real robot experiments + projectile-physics model.

**Learning method**: Hybrid — analytic free-flight projectile model (impulse-momentum decomposition) supplies a structured prior; regression-based learning fits residual unmodeled effects. Decouples translational release velocity from parasitic angular velocity by motion-design choice, then learns the residual.

**Action representation**: Release linear-velocity vector + release angular-velocity vector (impulse-momentum parameters of the throw). The throwing motion is parameterized to control these independently.

**Why cited in the surveys**: A 2025 throwing analog whose impulse-momentum decoupling could transfer to whip-handle motion: separate the linear release velocity (sets tip arrival point) from a wrist-flick angular impulse (sets traveling-wave energy down the rope).

**Key result (if any)**: Reaches target 6-DoF pose within (±5 cm, ±45°) in dozens of trials per pose; 40% sample-complexity reduction vs end-to-end RL on unseen pose targets; 70% transfer-learning speedup when reusing the in-hand-spinning skill on a new object with center-of-mass shift.
