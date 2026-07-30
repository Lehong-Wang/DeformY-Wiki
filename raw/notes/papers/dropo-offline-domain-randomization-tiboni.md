---
title: "DROPO: Sim-to-Real Transfer with Offline Domain Randomization"
authors: ["Gabriele Tiboni", "Karol Arndt", "Ville Kyrki"]
venue: "Robotics and Autonomous Systems"
year: 2023
arxiv_id: "2201.08434"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# DROPO: Offline Bayesian Sim Parameter Distribution from Real Rollouts

**One-line gist**: Infer a *distribution* over simulator physical parameters (not a point estimate) from a small pre-collected real dataset, then use that randomization distribution to train a policy that transfers zero-shot to the real robot.

**Task/Method setup**: Given a handful of offline real trajectories (no interactive data collection), DROPO maximises the likelihood of those trajectories under a *stochastic* simulator parameterised by a distribution φ over physical parameters (masses, friction, damping, etc.). A Bayesian optimisation loop searches φ-space; the resulting distribution is used as the domain randomisation range for downstream policy training. No point estimate is committed to; uncertainty is preserved and pushed into the randomisation.

**Sim vs real**: Pure sim-to-real; real data is used *only* for calibrating φ, not for policy training. A few minutes of pre-collected rollouts suffice (one-time, offline).

**Core idea / mechanism**:
- Likelihood of real trajectories under a parameter distribution is estimated by Monte-Carlo rollouts in simulation.
- Bayesian Optimisation (BO) over the distribution parameters maximises this likelihood.
- The resulting φ encodes *epistemic uncertainty* about the real system; training under this distribution naturally hedges the learned policy.
- No point-estimate sim-parameter needed; avoids the "sim optimism" failure mode where a perfectly tuned single sim misleads the policy.

**Why it matters for OUR problem**:
- *Sim2real for rope/DLO forward model*: Our meta-learned forward model (RMA-style) needs a good sim prior. DROPO's offline approach fits perfectly — a few-minute real calibration session (rope whipping, passive swings) can be used to infer a distribution over rope stiffness, damping, and mass without committing to a single wrong value.
- *Anti model-exploitation*: Calibrating a *distribution* rather than a point estimate directly produces a probabilistic ensemble-friendly sim prior, reinforcing robustness in our PETS-style planner against model exploitation.
- *One-time calibration constraint*: DROPO is explicitly designed for the one-time, per-rope offline regime — matching our hard constraint of no per-target adaptation.
- *Meta-learning synergy*: The inferred φ distribution is a natural prior for the context encoder; the real calibration rollouts can seed the RMA context, separating sim-prior calibration from per-deployment adaptation.

**Key result**: On MuJoCo locomotion and robotic manipulation benchmarks, DROPO outperforms ADR and other adaptive domain randomisation baselines in zero-shot sim-to-real transfer, using only ~20–50 offline real trajectories. Recovers physically meaningful parameter distributions and generalises without any online fine-tuning.
