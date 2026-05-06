---
title: "Real-to-Sim Robot Policy Evaluation with Gaussian Splatting Simulation of Soft-Body Interactions"
authors: "Kaifeng Zhang, Shuo Sha, Hanxiao Jiang, Changxi Zheng, Yunzhu Li (Columbia + SceniX + Google DeepMind)"
venue: "ICRA 2026"
year: 2026
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-2, l2c-4]
---

# Real-to-Sim Robot Policy Evaluation with Gaussian Splatting Simulation of Soft-Body Interactions

**One-line gist**: Real-demo-trained imitation policies (diffusion-policy class) evaluated in Gaussian-splatting + soft-body physics replicas of real scenes; tasks include rope routing, plush-toy packing, T-block pushing.

**Task setup**: Three deformable manipulation tasks — plush toy packing, rope routing, T-block pushing — used as a benchmark for real-to-sim policy evaluation.

**Sim vs real**: Real demonstrations train imitation policies; Gaussian-splatting reconstructions of the real scene are used as the simulator for evaluating those policies.

**Learning method**: SOTA imitation learning policies (e.g., diffusion policy) trained on real demos; the contribution is the Gaussian-splatting + soft-body physics evaluator, not a new policy class.

**Action representation**: End-effector trajectory inherited from the imitation policy class.

**Why cited in the surveys**: Inverts the sim2real assumption: train on real, evaluate in (high-fidelity) sim. Cited as a 2026 entry in the visually-faithful real-to-sim line; rope task is "routing" not "whipping" (quasi-static).

**Key result (if any)**: Faithful policy ranking matching real-world rollouts. https://real2sim-eval.github.io/
