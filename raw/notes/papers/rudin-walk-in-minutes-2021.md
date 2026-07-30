---
title: "Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning"
authors: [Nikita Rudin, David Hoeller, Philipp Reist, Marco Hutter]
venue: CoRL
year: 2021
arxiv_id: "2109.11978"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# Walk in Minutes: Massively Parallel RL

**One-line gist**: Train locomotion policies in minutes by running thousands of parallel simulated robots on a single GPU, paired with a game-inspired curriculum.

**Task/Method setup**: Quadruped (ANYmal) locomotion on flat and rough terrain. Policy trained entirely in sim via PPO with 4096–8192 parallel environments on one GPU. Action space = desired joint positions at 50 Hz; observation = proprioceptive state. No exteroception.

**Sim vs real**: Pure sim training; direct sim-to-real transfer. Flat-terrain policy trains in under 4 min; rough-terrain in ~20 min. Deployed zero-shot on hardware.

**Core idea / mechanism**:
- Isaac Gym GPU-accelerated physics: all env steps, resets, and reward computation stay on-device, eliminating CPU bottleneck.
- Game-inspired curriculum: terrain difficulty adapts per-robot based on reward signal, driving continual progression.
- Lightweight MLP policy (not recurrent); no domain randomization stack required beyond basic parameter ranges.
- Fast wall-clock convergence comes entirely from parallelism — same sample count as prior work, far fewer wall-clock seconds.

**Why it matters for OUR problem**:
- **Sim throughput**: The GPU-parallel recipe (Isaac Gym / similar) is the direct enabler for training our forward model ensemble over thousands of rope configurations simultaneously. Running 4k+ parallel rope swings per training step is the same architectural bet.
- **Sim2real / calibration**: Their zero-shot transfer relies on domain randomization; our meta-learned context encoder (RMA-style) replaces brute-force DR with a targeted real-calibration step — complementary, not competing.
- **Compact action**: Their fixed-frequency joint-position targets are the analogue of our spline via-points; massively parallel rollouts evaluate many trajectory candidates cheaply, supporting PETS-style ensemble planning.
- **Curriculum**: Their adaptive difficulty curriculum translates naturally to progressively harder tip-target positions / directions during forward-model training.

**Key result**: ANYmal walks on flat terrain after <4 min training; rough terrain in <20 min on a single workstation GPU. Direct sim-to-real transfer without fine-tuning.
