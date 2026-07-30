---
title: "Is Conditional Generative Modeling All You Need for Decision Making?"
authors: [Anurag Ajay, Yilun Du, Abhi Gupta, Joshua B. Tenenbaum, Tommi S. Jaakkola, Pulkit Agrawal]
venue: ICLR 2023 (Oral)
year: 2023
arxiv_id: "2211.15657"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# Decision Diffuser

**One-line gist**: Replace RL with a return-/constraint-conditioned diffusion model over trajectory space; planning becomes posterior sampling, not dynamic programming.

**Task/Method setup**: Offline RL on D4RL locomotion and AntMaze benchmarks. A diffusion model is trained to generate full action-sequence trajectories conditioned on target return (or constraint satisfaction). At test time, sampling with the desired return scalar produces a high-reward trajectory without any value-function or Q-learning.

**Sim vs real**: Simulation only (MuJoCo); no real-robot transfer reported.

**Core idea / mechanism**:
- Model p(τ | R) where τ is a full trajectory and R is a desired return, using DDPM-style denoising over the joint (s, a) sequence.
- Classifier-free guidance lets the model interpolate between unconditional and strongly return-conditioned samples at inference time.
- Constraints (e.g., avoid a region) are folded in as additional conditioning signals without retraining — just change the guidance mask.
- No Bellman backup, no critic, no importance weighting; the entire policy is the diffusion sampler.

**Why it matters for OUR problem**:
- **Robust planning**: Diffuser-style cost-guided diffusion is one of our two listed planning backends. This paper is the direct return-conditioning upgrade — shows how to steer a trajectory diffusion model toward a goal (tip position + arrival direction) at inference time without retraining.
- **Compact action / whole-trajectory generation**: Generates the entire trajectory in one shot (non-autoregressive), exactly matching our "forward model predicts whole tip trajectory" design choice.
- **Anti-exploitation**: Classifier-free guidance with tunable guidance strength acts as a trust-region: cranking up guidance too far overshoots; the paper characterizes this trade-off, directly relevant to our ensemble-pessimism / trust-region constraint.
- **Constraint conditioning**: Multi-signal conditioning (return + constraints) maps cleanly to our (target position, arrival direction) dual objective without architectural changes.

**Key result**: Matches or exceeds Decision Transformer and IQL on D4RL locomotion; large gains on AntMaze long-horizon tasks where sparse rewards defeat value-based methods. Guidance strength is a single scalar swept at test time.
