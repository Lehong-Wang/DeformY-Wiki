---
title: "DeXtreme: Transfer of Agile In-Hand Manipulation from Simulation to Reality"
authors: ["Ankur Handa", "Arthur Allshire", "Viktor Makoviychuk", "Aleksei Petrenko", "Ritvik Singh", "Jingzhou Liu", "Denys Makoviichuk", "Karl Van Wyk", "Alexander Zhurkevich", "Balakumar Sundaralingam", "Yashraj Narang", "Jean-Francois Lafleche", "Dieter Fox", "Gavriel State"]
venue: "ICRA 2023"
year: 2023
arxiv_id: "2210.13702"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# DeXtreme: Agile Dexterous Manipulation via Massive DR + GPU Sim

**One-line gist**: Train a dexterous in-hand cube-reorientation policy entirely in GPU-accelerated Isaac Gym with heavy automatic domain randomization (ADR), then transfer zero-shot to a real Allegro Hand.

**Task/Method setup**: Allegro Hand (16-DOF) must reorient a cube to arbitrary target orientations; policy is a proprioception + object-pose MLP trained with PPO; no task-specific reward shaping beyond orientation error. Massively parallel rollouts (~16k envs) in Isaac Gym. Automatic Domain Randomization (ADR) progressively expands randomization ranges for physics params, delays, observation noise, and visual appearance.

**Sim vs real**: Pure sim training; zero-shot transfer to real hardware. Real-time object pose from an onboard pose estimator (trained separately). No fine-tuning on real data; calibration is not required per target.

**Core idea / mechanism**: ADR — start with narrow randomization, auto-widen ranges as policy succeeds, forcing the policy to become robust to a large parameter envelope. Combines with asymmetric actor-critic (privileged sim state for critic only) and a learned pose estimator that outputs belief state from wrist camera + fingertip sensors.

**Why it matters for OUR problem**:
- **Sim2real / meta-adaptation**: Demonstrates that a one-time ADR-based sim training can achieve zero-shot real transfer for contact-rich dynamics — directly analogous to our goal of calibrating a meta-learned forward model once per rope then running open-loop.
- **Forward model robustness**: The ADR curriculum is a strong prior for how to shape a simulator ensemble to cover real physics variation — maps to our PETS-style ensemble pessimism strategy.
- **Compact action**: Policy outputs joint-torque deltas at 20 Hz; not spline-based but the asymmetric actor-critic (privileged critic) trick can be imported into our spline-decoder training to give the critic access to full sim state.
- **Anti model-exploitation**: ADR's progressive boundary expansion is an explicit mechanism against overfit to a single simulator mode — the dominant risk we flag in robust planning.

**Key result**: Matches or exceeds OpenAI Dactyl on cube reorientation while using only cheap RGB-D + proprioception (no motion capture at test time); >90% success on real Allegro Hand across diverse target orientations, zero-shot from sim.
