---
title: "DisMech: A Discrete Differential Geometry-Based Physical Simulator for Soft Robots and Structures"
authors: "Andrew Choi, Ran Jing, Andrew Sabelhaus, Mohammad Khalid Jawed"
venue: "IEEE RA-L 2024"
year: 2024
arxiv_id: "2311.18126"
doi: null
note_type: bibliography_only
sources: [report-1, report-2, l1e-5, l2d-1]
---

# DisMech — DDG-based DER Simulator for Soft Robots, Rods, and Shells

**One-line gist**: Discrete-differential-geometry physical simulator with Bergou-style DER rods + discrete elastic shells, implicit time integration (Newton solve at each step), claimed ~10× speedup over prior DER simulators while remaining accurate.

**Task setup**: Simulator capable of modeling soft robots, rods, shells, and combined structures. Includes RL companion repos.

**Sim vs real**: Validated against soft-robot hardware; not real-time real-deploy itself.

**Learning method**: Inverse-design / trajectory-opt via gradients through the implicit solver. Companion `QuantuMope/dismech-rl` repo provides RL examples (SAC, 500 parallel envs) including dynamic end-effector target-following and 3D obstacle reach.

**Action representation**: N/A (simulator) — but the RL examples use joint/EE actions.

**Why cited in the surveys**: Cited as one of the highest-fidelity open-source DER simulators with RL hooks. The "RL-trained policy for dynamic end-effector target following" companion example is the closest open-source analog to whip-tip-target-reaching.

**Key result (if any)**: ~10× speedup over prior DER simulators while keeping physical accuracy. Code: https://github.com/StructuresComp/dismech-rods (C++); https://github.com/StructuresComp/dismech-python (Py-DiSMech); https://github.com/StructuresComp/dismech-matlab (MAT-DiSMech).
