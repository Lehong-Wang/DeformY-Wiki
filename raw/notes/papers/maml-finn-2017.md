---
title: "Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks"
authors: [Chelsea Finn, Pieter Abbeel, Sergey Levine]
venue: ICML
year: 2017
arxiv_id: "1703.03400"
doi: "10.48550/arXiv.1703.03400"
note_type: bibliography_only
sources: [field-research]
---

# MAML — gradient-based meta-learning

**One-line gist**: Train a shared initialization such that a few gradient steps on a small task-specific dataset yield a well-adapted model, without modifying the architecture.

**Task/Method setup**: Meta-learning across a distribution of tasks; inner loop = K-shot gradient updates per task; outer loop = optimizes initialization so inner-loop adaptation succeeds. Model-agnostic: works for any differentiable model (regression, classification, RL policy).

**Sim vs real**: Not sim-specific, but demonstrated on few-shot RL locomotion tasks where adaptation uses real rollouts — directly analogous to sim-pre-train + real-finetune.

**Core idea / mechanism**: Two-level optimization. Outer: θ* = argmin Σ_task L(θ − α∇L(θ)). Inner: one or few gradient steps on task loss. Gradients flow through the inner update (second-order), so the initialization is explicitly shaped to be fast-adaptable. ProtoMAML / FOMAML (first-order approx) drops second-order terms for efficiency.

**Why it matters for OUR problem**:
- **Meta-adaptation (sim prior + real specialization)**: MAML is the canonical recipe for initializing a forward model on diverse sim rope/wind conditions, then fine-tuning to the real rope in a few-minute calibration session — directly maps to the RMA-style context-encoder approach in our design.
- **Forward model**: The meta-learned init gives a compact starting point; inner-loop adaptation corrects sim2real gap in rope stiffness/damping without full retraining.
- **Robust planning**: A well-adapted probabilistic ensemble (PETS) benefits from MAML initialization because each ensemble member shares the same fast-adapt prior, reducing variance in the adapted posteriors used for pessimistic planning.
- **Zero-shot after calibration**: MAML calibration is one-time per rope; at test time no further adaptation occurs — matches our HARD CONSTRAINT of no online adaptation per target.

**Key result**: MAML achieves few-shot adaptation in 1–5 gradient steps across regression, image classification (Omniglot/miniImageNet), and RL locomotion; outperforms task-agnostic fine-tuning and meta-learners without explicit gradient shaping.
