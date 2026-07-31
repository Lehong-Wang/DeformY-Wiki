---
title: "Model-Based Planning for Manipulation"
tags: [model-based-reinforcement-learning, planning, dynamics-model, world-model, trajectory-optimization, offline-planning, diffusion-planning, robot-learning]
key_venues: [NeurIPS, ICML, ICLR, CoRL, RSS, ICRA]
related_topics:
  - "[[dynamic-dlo-tip-targeting]]"
  - "[[sim-to-real-and-rapid-adaptation]]"
  - "[[dynamic-deformable-object-simulation]]"
key_people:
  - "[[sergey-levine]]"
  - "[[chelsea-finn]]"
  - "[[michael-janner]]"
  - "[[kurtland-chua]]"
key_papers:
  - "[[deep-reinforcement-learning-handful-trials-using]]"
  - "[[planning-diffusion-flexible-behavior-synthesis]]"
  - "[[learning-adapt-dynamic-real-world-environments]]"
  - "[[self-curriculum-model-based-reinforcement-learning]]"
  - "[[ropedreamer-kinematic-recurrent-state-space-model]]"
  - "[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]"
  - "[[mmp-motion-manifold-primitives-parametric-curve]]"
  - "[[differentiable-motion-manifold-primitives-reactive-motion]]"
  - "[[diffusion-policy-visuomotor-policy-learning-action]]"
linked_ideas: []
---

## Overview

This topic covers the paradigm at the heart of the research direction: **learn a forward/dynamics model of the rope-arm system, then plan offline against it** — rather than learning a model-free policy directly. A learned forward model turns tip-targeting into a search problem solvable with sampling-based planners (CEM, random shooting), diffusion-based trajectory generators, or world-model latent rollouts, and it is exactly the substrate a meta-learned per-object model would plug into. The canonical model-based RL recipe is **[[deep-reinforcement-learning-handful-trials-using]]** (PETS): a probabilistic ensemble of dynamics models with trajectory-sampling uncertainty propagation, planned via the cross-entropy method for extreme sample efficiency. **[[planning-diffusion-flexible-behavior-synthesis]]** (Diffuser) reframes planning as conditional generation, sampling whole trajectories from a diffusion model with classifier guidance — a template directly instantiated for DLOs by [[dynamics-informed-diffusion-policy]]. Deformable-specific instances include **[[self-curriculum-model-based-reinforcement-learning]]** (imagined-rollout goal curriculum for DLO shape control), **[[ropedreamer-kinematic-recurrent-state-space-model]]** (a quaternionic Dreamer-style world model for rope dynamics), and the differentiable-physics planning surface of **[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]**. A recurring concern across this topic is **robustness to model exploitation**: a planner optimizing against an imperfect learned model can drive the system into regions where the model is confidently wrong, motivating uncertainty-aware ensembles, physics priors, and compact action spaces.

## Timeline

| Year | Anchor result | What changed |
|---|---|---|
| 2018 | PETS | Probabilistic ensembles + trajectory sampling make model-based RL competitive with model-free at a fraction of the samples |
| 2019 | GrBAL / ReBAL | Online meta-learned dynamics adaptation lets a learned model re-fit within a few timesteps of new dynamics |
| 2022 | Diffuser | Planning reframed as diffusion sampling over whole trajectories with flexible test-time guidance |
| 2026 | RopeDreamer / Self-Curriculum-MBRL / DIDP | World-model and MBRL recipes specialized to deformable linear objects; diffusion-as-planner instantiated for 3D rope whipping |

## Seminal works

- **[[deep-reinforcement-learning-handful-trials-using]]** (Chua et al., NeurIPS 2018, PETS) — probabilistic ensemble dynamics + [[trajectory-sampling-uncertainty-propagation]] planned with CEM; the reference recipe for sample-efficient model-based control.
- **[[planning-diffusion-flexible-behavior-synthesis]]** (Janner et al., ICML 2022, Diffuser) — establishes [[planning-as-diffusion]]: trajectory-level generative planning with classifier-guided test-time objectives.
- **[[mmp-motion-manifold-primitives-parametric-curve]]** (Lee, T-RO 2024, MMP++/IMMP++) — planning as constrained optimization over a 3–6-dim latent state $(z,\tau)$ of a learned trajectory manifold ([[motion-manifold-primitives]]): in-distribution constraint replaces a dynamics model, 10 Hz replanning around moving obstacles, 3–4 orders of magnitude faster than RRT-Connect.
- **[[diffusion-policy-visuomotor-policy-learning-action]]** (Chi et al., RSS 2023 / IJRR 2024) — the policy-side counterpart of [[planning-as-diffusion]]: conditional $p(A|O)$ with FiLM conditioning instead of joint $p(A,O)$ with inpainting, which is what makes diffusion viable inside a real-time visual control loop. Listed here because [[planning-diffusion-flexible-behavior-synthesis]] and [[dynamic-manipulation-deformable-objects-3d-simulation]] both sit in this topic and neither is legible without it.
- **[[differentiable-motion-manifold-primitives-reactive-motion]]** (Lee, ICRA 2026, DMMP) — offline manifold + online sample-and-verify as the planning substrate: constraint satisfaction is amortized into the generator via [[trajectory-manifold-optimization]], yet a 0.2 s physics verifier is *still* required at deployment and dominates runtime. The clearest published statement that amortization does not remove the verifier.

## SOTA tracker

- **[[ropedreamer-kinematic-recurrent-state-space-model]]** (TU Darmstadt + Honda 2026) — [[quaternionic-rssm-dlo]] world model preserving rope inextensibility by construction; 40.5% rollout-error reduction over a baseline RSSM.
- **[[self-curriculum-model-based-reinforcement-learning]]** (Tsinghua 2026) — [[self-curriculum-goal-generation]] MBRL + Jacobian visual servoing; 30/30 zero-shot real-world success on DLO shape control.
- **[[dynamic-manipulation-deformable-objects-3d-simulation]]** (DIDP) — [[dynamics-informed-diffusion-policy]] with [[physics-informed-test-time-adaptation]]; the leading diffusion-planning result on 3D rope whipping (cross-listed under [[dynamic-dlo-tip-targeting]]).

## Key benchmarks

- **[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]** (ICLR 2023 Oral) — JAX differentiable-physics benchmark with a built-in WhipRope task; reports APG 0.83 vs SHAC 0.66 vs PPO 0.25.
- Standard continuous-control suites (HalfCheetah/Ant/etc.) underpin the PETS and Diffuser evaluations.

## Open problems

### Known gaps

- **Model exploitation under planning**: optimizers can exploit errors in a learned forward model; uncertainty-aware ensembles (PETS) and physics priors mitigate but do not eliminate this.
- **Long-horizon DLO rollouts**: error compounds over the many timesteps a fast whip spans, limiting purely-learned world models for high-speed tip targeting.

### Methodological gaps

- **Forward model + offline planner for free-tip 3D targeting**: the cleanest formulation of the research direction — a meta-learned per-object forward model plus robust offline planning — has not been demonstrated end-to-end on real rope whipping.
- **Compact, exploitation-resistant action spaces**: how to plan over a low-dimensional action parameterization (see [[compact-action-parameterization]]) so the planner cannot find degenerate model-exploiting trajectories.

## Concepts
- [[online-meta-learned-dynamics-adaptation]]
- [[physics-informed-action-prior]]
- [[physics-informed-test-time-adaptation]]
- [[trajectory-sampling-uncertainty-propagation]]
- [[quaternionic-rssm-dlo]]
- [[self-curriculum-goal-generation]]
- [[planning-as-diffusion]]
- [[dynamics-informed-diffusion-policy]]
- [[planner-generated-motion-corpus]] — use a slow classical planner offline to manufacture the corpus a fast amortizer is then trained on.
