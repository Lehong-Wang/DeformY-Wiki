---
name: "Junyi Cao"
affiliation: "University of Massachusetts Amherst"
research_areas: [differentiable-simulation, deformable-linear-object-manipulation, physics-based-learning, robot-learning, neural-material-modeling]
date_updated: 2026-07-30
type:
  kind: researcher
---

## Research areas

- Differentiable physics simulation for robotics (Taichi / Genesis-based solvers)
- Deformable linear object (rope, cable, wire) modeling and manipulation
- Neural and hybrid material models identified from real observations
- Benchmark and simulation-platform construction for robot skill learning

## Recent work

- [[dlo-lab-benchmarking-deformable-linear-object]] (ICML 2026) — first and corresponding author.
  Built the customized differentiable Discrete-Elastic-Rods solver inside Genesis (bending
  plasticity, loop topology, two-way rigid and MPM coupling) and the 10-task DLO-Lab benchmark.

## My notes

Relevant to this wiki as the maintainer of the most credible **independent** differentiable DLO
simulator — the natural second verifier for
[[sim-stage-c-robustness-and-verifier-mismatch]], since the DLO-Lab stack (Genesis + Taichi DER +
symplectic Euler + PBD contact) shares no code and no discretization with the project's primary
Isaac Lab + Cosserat-rod stack. Prior work on neural material identification from video (NeuMA)
indicates a consistent interest in closing the sim-real parameter gap with gradients, which is the
same lever DLO-Lab uses for its rope system identification.
