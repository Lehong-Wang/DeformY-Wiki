---
title: "Data-Efficient Robotic Object Throwing with Model-Based Reinforcement Learning (MC-PILOT)"
authors: "Niccolò Turcato, Giulio Giacomuzzo, Matteo Terreran, Davide Allegro, Ruggero Carli, Alberto Dalla Libera"
venue: "arXiv (under review)"
year: 2025
arxiv_id: "2502.05595"
doi: null
note_type: bibliography_only
sources: [report-1, report-2, l1c-2]
---

# MC-PILOT: Data-Efficient Robotic Object Throwing with Model-Based RL

**One-line gist**: PILCO-style probabilistic MBRL with explicit release-delay model; adapts to new objects/3D targets with few real-Panda rollouts.

**Task setup**: Pick-and-throw of objects from a bin to user-specified 3D target locations on a Franka Emika Panda. Generalizes to unseen targets with limited additional rollouts.

**Sim vs real**: Both — extensive sim experiments + real Panda deployment.

**Learning method**: Monte-Carlo PILCO-style probabilistic MBRL. Learns a Gaussian-process / data-driven dynamics model that explicitly models (i) release-state uncertainty and (ii) release-time delay of the gripper. Policy optimized via stochastic gradient through Monte-Carlo rollouts. Includes an explicit release-delay estimation module in the RL loop.

**Action representation**: Release state — release end-effector position + release velocity vector — at the moment the gripper opens. The rest of the swing is a parameterized open-loop trajectory whose end-state matches the release-state. Goal is a 3D point input.

**Why cited in the surveys**: Closest 2025 throwing analog to the rope-tip-target problem because of the explicit release-delay model — directly transferable to whip-tip timing where the tip lags the handle by a stiffness-dependent delay.

**Key result (if any)**: Outperforms model-free RL and analytic ballistic baselines; adapts to new objects/targets with substantially fewer interactions.
