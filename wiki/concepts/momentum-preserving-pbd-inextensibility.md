---
title: "Momentum-Preserving PBD Inextensibility"
aliases: ["mass-weighted PBD correction", "momentum-conserving inextensibility enforcement", "momentum-preserving position-based inextensibility"]
tags: [DLO, simulation, position-based-dynamics, inextensibility, momentum-conservation, discrete-elastic-rods]
maturity: emerging
key_papers: ["[[deform-differentiable-discrete-elastic-rods-real]]"]
first_introduced: "2024"
date_updated: 2026-05-06
related_concepts: []
parent_topic: "[[dynamic-deformable-object-simulation]]"
---

## Definition

Momentum-preserving PBD inextensibility is a modification of the standard [[position-based-dynamics]] (PBD) inextensibility constraint enforcement, in which the position correction applied to each pair of successive vertices on a deformable linear object (DLO) is **scaled by a mass-ratio coefficient** $\beta^i$ chosen so that the corrections, summed across the rod, satisfy both **linear** and **angular** momentum conservation. In vanilla PBD, the paired correction $\Delta \hat{\textbf{x}}^i = -\Delta \hat{\textbf{x}}^{i+1}$ violates momentum conservation as soon as vertex masses differ; the modification restores it without changing the constraint's quasi-static enforcement quality.

## Intuition

PBD enforces a length constraint between successive vertices by projecting them onto the constraint manifold along the edge direction. When all vertices have equal mass, this symmetric projection conserves momentum trivially. Real DLOs have non-uniform mass (markers, end attachments, varying density along the cable), and a symmetric projection then biases the correction toward the lighter vertex, producing a small but accumulating linear and angular momentum drift each step. For dynamic motions — swinging, casting, throwing — the drift compounds and the simulator visibly decelerates or rotates the rod artificially. The fix is to redistribute the correction in inverse proportion to the masses, so the heavier vertex moves less than the lighter one: $\beta^i = \|\textbf{M}^{i+1}\| / (\|\textbf{M}^i\| + \|\textbf{M}^{i+1}\|)$. This is a one-line change that makes momentum exactly conserved.

## Formal notation

For successive vertices $\hat{\textbf{x}}^i$, $\hat{\textbf{x}}^{i+1}$, define the inextensibility constraint

$$\mathrm{C}(\hat{\textbf{x}}^i_{t+1}, \hat{\textbf{x}}^{i+1}_{t+1}) = \big| \|\hat{\textbf{x}}^i_{t+1} - \hat{\textbf{x}}^{i+1}_{t+1}\|_2 - \|\bar{\textbf{e}}_i\|_2 \big|.$$

Vanilla PBD applies the correction

$$\Delta \hat{\textbf{x}}^i_{t+1} = \mathrm{C}(\hat{\textbf{x}}^i_{t+1}, \hat{\textbf{x}}^{i+1}_{t+1}) \cdot \frac{\hat{\textbf{x}}^{i+1}_{t+1} - \hat{\textbf{x}}^i_{t+1}}{\|\hat{\textbf{x}}^{i+1}_{t+1} - \hat{\textbf{x}}^i_{t+1}\|_2}.$$

The momentum-preserving variant scales the correction:

$$\Delta \hat{\textbf{x}}^i_{t+1} \leftarrow \beta^i \, \Delta \hat{\textbf{x}}^i_{t+1}, \qquad \Delta \hat{\textbf{x}}^{i+1}_{t+1} \leftarrow -\beta^{i+1} \, \Delta \hat{\textbf{x}}^i_{t+1},$$

with $\beta^i = \|\textbf{M}^{i+1}\| / (\|\textbf{M}^i\| + \|\textbf{M}^{i+1}\|)$, $\beta^{i+1} = \|\textbf{M}^i\| / (\|\textbf{M}^i\| + \|\textbf{M}^{i+1}\|)$. Then

$$\sum_i \textbf{M}^i \cdot \Delta \hat{\textbf{x}}^i_{t+1} = \textbf{0} \quad \text{and} \quad \sum_i \textbf{r}^i \times (\textbf{M}^i \cdot \Delta \hat{\textbf{x}}^i_{t+1}) = \textbf{0}.$$

## Variants

- **Per-edge** vs. **chain-wide**: DEFORM applies the correction per successive vertex pair (Gauss-Seidel style); chain-wide formulations exist but require global solves.
- **Iterative refinement**: apply the correction to convergence ($\mathrm{C} < \epsilon$); typically converges in a few iterations for DLOs.
- **Coupling with residual physics**: the correction is applied **after** any [[neural-residual-on-physics-model]] DNN step, so the DNN cannot learn to violate inextensibility.

## Comparison

vs. **vanilla PBD inextensibility**: same constraint manifold, but vanilla PBD violates momentum whenever vertex masses are non-uniform.

vs. **XPBD** (extended PBD): XPBD adds compliance and a Lagrange-multiplier formulation for stiffness control but does not fix the momentum issue without an analogous mass-weighting modification.

vs. **stiff penalty / mass-spring**: penalty methods enforce inextensibility via a stiff spring, which destabilizes the time integrator and conserves momentum only approximately.

vs. **constraint manifold projection** (e.g., Lagrangian projection): mathematically equivalent in the limit, but PBD-style projection is much faster per step.

## When to use

- Simulating dynamic, swinging DLOs with non-uniform mass distribution (markers, end caps, varying density).
- Inside a differentiable simulator, where the constraint correction must be a smooth differentiable function of vertex positions.
- When **momentum drift** in the inextensibility step is an observable failure mode of the existing simulator, especially for compliant DLOs (DEFORM ablation: largest gain on the most compliant cable).

## Known limitations

- Assumes pairwise edge constraints; a chain of corrections is solved Gauss-Seidel, which can require multiple sweeps for tight convergence.
- Does not handle **contact** corrections jointly with inextensibility — separate handling is needed.
- The momentum proof is for the case where corrections are applied to internal (non-grasped) vertices; boundary handling at grippers is heuristic.
- Empirically validated only on rope-like DLOs in DEFORM; cloth or sheet generalizations would need the appropriate constraint family.

## Open problems

- **Joint inextensibility + self-contact** correction with momentum conservation, needed for knot-tying.
- **Adaptive iteration count**: how many Gauss-Seidel sweeps are needed depends on the DLO state; an analytic stopping criterion would help real-time deployment.
- Integration with **GPU-batched** simulators where Gauss-Seidel ordering becomes a parallelization bottleneck.

## Key papers

- [[deform-differentiable-discrete-elastic-rods-real]] — DEFORM (Chen et al., CoRL 2024) introduces and validates the mass-ratio scaling, with an ablation showing the largest gain on the most compliant DLO (DLO 1).

## My understanding

This is one of the cleanest small contributions of the DEFORM paper: a one-equation fix to a known PBD shortcoming, with a clear momentum-conservation proof and a clean ablation showing the effect is biggest exactly where the theory predicts (compliant, dynamically swinging DLOs). The pattern likely transfers to any setting where PBD is used on a non-uniform-mass mechanical system — articulated soft bodies, slender filaments with localized inertia, etc. The companion engineering question is whether the same mass-weighting trick admits a closed-form chain-wide solve that avoids Gauss-Seidel iteration; this is open.
