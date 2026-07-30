---
title: "BayesSim: Adaptive Domain Randomization via Probabilistic Inference for Robotics Simulators"
authors: ["Fabio Ramos", "Rafael Possas", "Dieter Fox"]
venue: "RSS"
year: 2019
arxiv_id: "1906.01728"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# BayesSim: Posterior over Sim Params from Real Rollouts

**One-line gist**: Likelihood-free Bayesian inference (LFBI) computes a full posterior over simulator physical parameters from a small set of real robot trajectories, replacing blind uniform domain randomization with a data-informed distribution.

**Task/Method setup**: Given a black-box simulator and a handful of real state-action trajectories, BayesSim trains a conditional density network (mixture-density network with summary statistics) to approximate `p(θ_sim | real_data)` without evaluating the likelihood explicitly. The posterior is then used as the randomization distribution for policy or model training.

**Sim vs real**: Core loop — run sim with many θ samples to build a dataset of (summary_stats, θ) pairs; train MDN offline; at calibration time, pass a few minutes of real robot rollouts through the MDN to obtain the posterior. Supports iterative refinement (sequential BayesSim).

**Core idea / mechanism**: Summary statistics compress variable-length trajectories into fixed vectors; MDN maps those statistics to a Gaussian-mixture posterior over θ. The key insight is that the posterior collapses the large uniform prior to a tight, physically plausible region — eliminating the "over-randomization" failure mode where sim params far from reality dominate training.

**Why it matters for OUR problem**:
- **Meta-adaptation / sim2real**: Directly addresses the one-time per-rope calibration requirement. A few minutes of real rope swing data can be passed through BayesSim to infer posteriors over rope stiffness, damping, density — specializing the sim prior without per-target rollouts.
- **Forward model**: Training the forward model (action → tip trajectory) under the BayesSim posterior rather than a flat prior yields a model ensemble that is concentrated on physically consistent parameters, reducing epistemic uncertainty at deployment.
- **Robust planning**: PETS-style ensemble uncertainty is dominated by model spread; BayesSim shrinks that spread to the credible region, making pessimism/trust-region costs more informative and less conservative.
- **RMA-style context encoder**: BayesSim provides a principled way to pre-select the sim-parameter distribution the context encoder must adapt over, rather than hand-tuning randomization ranges.

**Key result**: On manipulation and locomotion tasks, BayesSim posterior-based randomization outperforms uniform DR in both sample efficiency and final policy performance; sequential refinement with 2-3 real rollout batches converges to near-oracle performance.
