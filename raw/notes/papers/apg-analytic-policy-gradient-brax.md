---
title: "Brax — A Differentiable Physics Engine for Large Scale Rigid Body Simulation (+ APG)"
authors: ["C. Daniel Freeman", "Erik Frey", "Anton Raichuk", "Sertan Girgin", "Igor Mordatch", "Olivier Bachem"]
venue: "NeurIPS Datasets and Benchmarks"
year: 2021
arxiv_id: "2106.13281"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# Brax / Analytic Policy Gradient (APG)

**One-line gist**: A massively parallel JAX physics engine that exposes analytic (backprop-through-sim) policy gradients; the APG variant is the top-performing RL method on the DaXBench WhipRope task.

**Task/Method setup**: Brax simulates articulated rigid-body dynamics in JAX and includes reimplementations of PPO, SAC, ES, and APG that compile alongside environments and run on GPU/TPU. APG directly backpropagates the policy-parameter gradient through the differentiable simulator rather than estimating it via Monte-Carlo rollouts.

**Sim vs real**: Simulator-only; no real-world experiments in the Brax paper itself. DaXBench (arXiv 2210.13066, ICLR 2023) uses a JAX position-based-dynamics rope/cloth/fluid simulator and reports that APG outperforms PPO and SHAC on the WhipRope task — the only benchmark where differentiable gradients decisively help.

**Core idea / mechanism**: Replace REINFORCE-style stochastic gradient estimation with exact first-order gradients `∂reward/∂θ` computed by autodiff through the simulation rollout. Requires the simulator to be (nearly) smooth; discontinuities (collisions, contacts) cause gradient variance but JAX's XLA compilation amortizes the cost over thousands of parallel envs.

**Why it matters for OUR problem**:
- *Forward-model*: APG is the direct analogue to our "learn forward model, then differentiate through it" planning loop — confirms that analytic gradients are feasible for rope dynamics when the model is differentiable.
- *Compact-action / spline decoder*: Backprop through the simulator makes end-to-end optimization of a smooth spline parameterisation tractable; gradient signal reaches the via-point parameters directly.
- *Robust planning*: The DaXBench WhipRope result (APG wins) validates that gradient-based trajectory optimisation beats black-box RL for tip-targeting, motivating our PETS/diffusion planning stage.
- *Sim2real / meta-adaptation*: Brax's massively parallel GPU rollouts are the natural backend for our sim-prior training; APG's gradient signal can also supervise a meta-learned context encoder during calibration.

**Key result**: On DaXBench WhipRope, APG achieves the highest task success among evaluated methods (PPO, SHAC, APG), demonstrating that differentiable simulation gives qualitatively better gradients for high-dimensional DLO tip control than sample-based RL.
