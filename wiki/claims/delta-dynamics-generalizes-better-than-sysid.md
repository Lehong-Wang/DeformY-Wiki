---
title: "Learning delta dynamics from observed trajectories generalizes better than explicit system identification for dynamic deformable manipulation"
slug: "delta-dynamics-generalizes-better-than-sysid"
status: weakly_supported
confidence: 0.7
tags: [DLO, sim-to-real, residual-learning, system-identification, delta-dynamics, robot-learning]
domain: "Robotics"
source_papers: ["[[iterative-residual-policy-goal-conditioned-dynamic]]"]
evidence:
  - source: "[[iterative-residual-policy-goal-conditioned-dynamic]]"
    type: supports
    strength: moderate
    detail: "On simulated extrapolation rope parameters, IRP reaches 1.5 cm tip-to-goal error in 16 iterations vs. ~5+ cm for SysID and SysID_GT (the latter given ground-truth length/density). On real-world ropes with un-modeled aerodynamics (long-cloth) and non-uniform mass distribution (bullwhip, knotted), IRP-9 achieves 1.9-2.6 cm vs. 8-22 cm for the OptSim oracle that uses *measured* rope parameters. The paper's interpretation: SysID-class methods are bottlenecked by the *predefined* parameter set (length, density), which cannot capture aerodynamic drag, friction, or stiffness anisotropy; the observed trajectory implicitly encodes all of these so the delta-dynamics network can absorb them without an explicit parameter for each."
conditions: "Parametric simulator that captures the dominant dynamics but not all real-world effects (aerodynamics, friction, contact); fully observable trajectory; repeatable action; adequate offline simulation budget for training the delta-dynamics network. The comparison is against SysID with a fixed predefined parameter set (length, density) and OptSim with manually measured parameters; it is not against learned latent-space sysid or differentiable simulators."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

For dynamic deformable manipulation tasks where (i) the simulator captures the dominant dynamics but is missing effects (aerodynamics, friction, contact) that matter at execution time, and (ii) the manipuland is fully observable, **predicting trajectory shifts from action perturbations conditioned on the observed trajectory** generalizes to real-world dynamics substantially better than **inferring an explicit physical-parameter vector and planning on the simulator with that vector**. The advantage is most pronounced on rope/cloth instances whose un-modeled effects (aerodynamic drag, non-uniform mass distribution, novel material) are far from the training distribution.

## Evidence summary

Two observations from the source paper jointly support the claim:

1. **Out-of-distribution simulation.** On extrapolation rope parameters (lengths and densities outside the training grid), IRP reaches ~1.5 cm tip-to-goal error vs. SysID baselines plateauing at higher error — even when SysID is given the ground-truth (length, density) of the test rope. This isolates the effect of the parameter set: SysID with the *correct* parameters is still worse than IRP because the parameter set is incomplete.

2. **Un-modeled real-world physics.** On real ropes with strong aerodynamic effects (long-cloth) or non-uniform mass distribution (bullwhip), OptSim — an oracle exhaustive search using *measured* (length, density, width) — reaches 14-22 cm error while IRP reaches 1.9 cm. The gap cannot be closed by better parameter estimation because the simulator's parameter set does not include the dominant un-modeled effects.

Together these establish that the bottleneck is **what the parameter vector cannot represent**, not how accurately the parameter vector is estimated. The observed trajectory carries this missing information; the delta-dynamics network is conditioned on it directly.

## Conditions and scope

The claim is restricted to:

- **Parametric simulators with known structural gaps** — the result has not been tested when the simulator already captures (e.g.) aerodynamics correctly.
- **Tasks with fully observable trajectories.** Partial observability invalidates the conditioning argument.
- **Deformable manipulation specifically.** The argument applies most strongly when the un-modeled effects (aerodynamics, friction, distributed inertia) interact non-linearly with action; rigid-body manipulation may not show the same advantage.
- **Comparison against parameter-set SysID and analytical OptSim.** Modern alternatives the paper does not compare against include:
  - Differentiable physics with end-to-end gradient-based parameter inference;
  - Latent-space SysID where the parameter representation is itself learned;
  - Real-world fine-tuning of a sysid-trained policy.

## Counter-evidence

None directly observed in the source paper. Plausible counter-stories:

1. **Latent-space SysID** that learns its own parameter representation might capture aerodynamics implicitly and close the gap.
2. **A higher-fidelity simulator** (e.g., the Cosserat-based DeformX) might shrink the un-modeled gap enough that explicit sysid + planning becomes viable. The DeformX paper supports this: PPO with a Cosserat backend transfers without iteration. In that regime, the IRP claim's *necessity* weakens, even if the claim's *correctness in its scope* is preserved.
3. **Domain randomization in the parameter set** might absorb the un-modeled effects implicitly without requiring delta dynamics — not directly tested.

## Linked ideas

(none yet)

## Open questions

- Does the advantage shrink to zero when the simulator becomes Cosserat-grade (DeformX) or full-FEM?
- Does delta dynamics still beat a *learned* (latent) sysid representation?
- How does the claim change for closed-loop visuomotor policies vs. open-loop primitives?
- For non-deformable dynamic tasks (juggling, throwing, hammering), does the same advantage hold or is it specific to deformables?
