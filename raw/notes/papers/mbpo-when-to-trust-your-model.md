---
title: "When to Trust Your Model: Model-Based Policy Optimization"
authors: [Michael Janner, Justin Fu, Marvin Zhang, Sergey Levine]
venue: NeurIPS 2019
year: 2019
arxiv_id: "1906.08253"
doi: "10.5555/3454287.3455188"
note_type: bibliography_only
sources: [field-research]
---

# MBPO: Short Model Rollouts with Ensemble Dynamics

**One-line gist**: Use short, branched model rollouts from real data (not long open-loop rollouts) to safely exploit a learned probabilistic ensemble dynamics model without compounding errors.

**Task/Method setup**: Model-based RL for continuous control (MuJoCo benchmarks). Learns a probabilistic ensemble of neural network dynamics models; generates synthetic transitions by branching short rollouts (horizon k) from real states, mixing them into a replay buffer for SAC policy optimization.

**Sim vs real**: Purely simulated benchmarks; real-world transfer not studied directly. The theoretical analysis applies generally to any model with bounded generalization error.

**Core idea / mechanism**:
- Proves that monotonic policy improvement is guaranteed when model error is bounded.
- Derives that model exploitation grows quadratically with rollout horizon k; hence, short rollouts (k = 1–5) maximally extract model value while keeping compounding error small.
- Ensemble disagreement serves as an implicit uncertainty signal; branching from real visited states keeps the model in-distribution.
- Achieves Dyna-style sample efficiency without the instability of long imagined trajectories.

**Why it matters for OUR problem**:
- **Forward model / robust planning**: MBPO's theoretical justification for short rollouts directly informs how to use our learned tip-trajectory forward model during planning. Long open-loop rollouts amplify model error — exactly the "model exploitation" anti-target in our robust cost-guided planning (PETS/Diffuser). Short, ensemble-checked rollouts or pessimistic ensemble aggregation follow from the same principle.
- **Ensemble uncertainty**: The probabilistic ensemble with disagreement-based pessimism is the backbone of PETS-style planning we plan to adopt; MBPO formalizes when ensemble members can be trusted.
- **Sim2real**: The branching-from-real-data strategy maps onto our calibration step — few real rollouts anchor the model in the real distribution, preventing sim-only exploitation.
- **Meta-adaptation**: MBPO's framing supports RMA-style context encoders: the context encoder shifts the prior model toward the real rope, and short rollouts remain valid under bounded residual error.

**Key result**: MBPO matches or exceeds model-free SAC with 20–100× fewer real environment samples on HalfCheetah, Hopper, Walker, Ant; rollout horizon ablation confirms k ≤ 5 optimal — longer horizons degrade performance due to compounding model error.
