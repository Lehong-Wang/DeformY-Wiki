---
name: "Siwei Chen"
affiliation: "National University of Singapore (AdaComp Lab)"
tags: [deformable-object-manipulation, differentiable-physics, JAX, MPM, RL, robot-learning, benchmark]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

- Deformable Object Manipulation (DOM): rope, cloth, liquid, elastoplastic.
- Differentiable physics for robot learning (JAX, analytic policy gradients, differentiable MPC).
- Benchmarking: building shared evaluation platforms for DOM algorithms across paradigms (planning, RL, IL).

## Key papers

- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — co-first author with Yiqing Xu and Cunjun Yu. ICLR 2023 Oral. Introduced DaXBench, the first multi-object differentiable DOM benchmark, plus the DaX JAX simulator.

## Recent work

- DaXBench (ICLR 2023 Oral) — DOM differentiable benchmark.

## Collaborators

- Yiqing Xu (NUS, AdaComp) — DaXBench co-first author.
- Cunjun Yu (NUS, AdaComp) — DaXBench co-first author.
- Linfeng Li (NUS) — DaXBench.
- Xiao Ma (Sea AI Lab) — DaXBench.
- Zhongwen Xu (Sea AI Lab) — DaXBench.
- David Hsu (NUS) — advisor; AdaComp lab head.

## My notes

NUS AdaComp Lab co-first author, distinct from "Yizhou Chen" (the DEFORM PyTorch-side differentiable DLO co-author). The DaXBench contribution sits on the JAX side of the differentiable-DOM ecosystem and is complementary to the DEFORM/PyTorch line: DaXBench treats rope as MLS-MPM particles for breadth (rope + cloth + liquid + elastoplastic in one benchmark), while DEFORM specializes in differentiable Cosserat / Discrete Elastic Rods for DLO physical fidelity. For DLO-specific work, both lines are the natural rivals to compare against.
