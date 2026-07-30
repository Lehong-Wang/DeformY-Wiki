---
name: "Conditional Flow Matching over Motion-Primitive Parameters"
slug: conditional-flow-matching-motion-parameters
type: architecture
tags: [flow-matching, conditional-generative-model, movement-primitives, amortized-inference, one-to-many-inverse, goal-conditioned, trajectory-generation, robot-learning]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts: []
date_updated: 2026-07-30
---

## Problem setting

Amortize the inverse map goal → action for a whole-trajectory action: given
g = (target position, arrival direction), emit swing parameters w whose rollout passes
through the goal. The map is **one-to-many** — several distinct swing families reach the same
(position, direction), and pass-time nuisance variation adds more — so a deterministic MSE
regressor trained on the raw hindsight pool would average across distinct families and emit
invalid *means*.

The amortizer of [[direction-conditioned-open-loop-rope-tip-targeting]], trained on the pool
produced by [[per-timestep-hindsight-relabeling]] over
[[smooth-basis-swing-parameterization]].

## Mechanism

A conditional flow-matching network models p(w | g): it represents the solution **set**
rather than a point estimate. Three properties matter here, in order of importance:

1. **Multi-candidate sampling is free** — exactly what
   [[sim-verified-best-of-n-selection]] consumes. A regressor gives one candidate; the
   verifier needs N.
2. **Fixing the noise seed while sweeping the goal yields continuous goal→motion families.**
   This is the "target-space continuity" property previously hoped for from an MMP++ latent —
   obtained here without building an autoencoder.
3. **Simplest modern choice** for the job; flow matching over ~30-D parameters conditioned on
   a 5-D goal is a small, well-conditioned learning problem.

**Prediction under test.** Direction-in-the-goal is expected to *reduce* multimodality
(conditioning largely selects the swing family: overhand ↓, sidearm →, underhand ↑), which
should make the problem *easier* than the position-only case, not harder. The corollary is
that **deterministic regression may be rehabilitated** — so B2 (regression) and B3 (flow) are
run head-to-head rather than assuming the generative head is necessary.

## Procedure

1. Train on the CVT-balanced, speed-stratified, pass-canonicalized hindsight pool, held out
   **at the rollout level** (a rollout and all its relabeled pairs live in one split — a
   cell-level split leaks near-identical neighbors into training).
2. Condition on g = (p*, d̂*) and, if pass time was not canonicalized away, on normalized
   pass time (sampled at deployment).
3. Sample N candidates per goal at inference; hand them to the verifier.
4. Report **blind top-1** alongside verified top-N. Blind top-1 is the honest measure of pure
   amortization, and the blind↔verified gap measures how much the pipeline leans on the
   simulator.
5. Optional GCSL loop: sample goals at high-error atlas cells → generate with noise → roll out
   → relabel → retrain.

Implementation stack: `torchcfm` (MIT) + `ALRhub/MP_PyTorch`, with
`ScheiklP/movement-primitive-diffusion` as the architectural reference. **No end-to-end base
repo exists** — DA-MMP and DMMP have released no code (re-checked 2026-07-28). Visuomotor
policy frameworks should be avoided; their abstractions all mismatch one-shot parameter
generation.

## Assumptions

- The hindsight pool covers the goal regions where claims will be made. Coverage is measured
  by [[direction-reachability-atlas]], not assumed.
- Pass-time nuisance variation has been canonicalized or explicitly conditioned on.
  Otherwise even flow matching wastes capacity modelling irrelevant trajectory suffixes.
- Enough candidates can be verified per goal for sampling to pay off — i.e. the declared
  per-goal sim budget is nonzero.

## Limitations

- **Verification is doing part of the work.** A large blind↔verified gap is acceptable in
  simulation but signals that real-robot transfer will lean heavily on calibrated-sim
  fidelity; it must be recorded as a known dependence, not smoothed over.
- Generative capacity may be unnecessary if H4 holds and regression matches it — in which
  case this arm loses the shootout on complexity grounds, which is a legitimate outcome.
- Learns only the *natural* correlation structure present in the pool; regions the sweep
  prior missed cannot be invented by the generator.
- Statistical floor: shootout decisions require non-overlapping Wilson intervals or a paired
  test on shared goals, at fixed eval-set size.

## Tradeoff profile

| Against | This method |
|---|---|
| Deterministic regression (B2) | Represents the solution set instead of averaging it, and supplies N candidates for verification; costs complexity and training time, and may be unnecessary if direction conditioning collapses the modes |
| Pool nearest-neighbour lookup (B1) | Smooth interpolation over a 5D goal space with ~10⁶–10⁷ useful cells, where lookup degrades; costs the exactness of a stored elite |
| Conditional manifold / DMMP-DA-MMP-style (B4) | No autoencoder stage and no latent-dimension tuning; forgoes an explicit latent density and the ability to search a compact latent at test time. Necessary-condition check on the manifold arm: an *unconditional* manifold needs latent dim ≥ 4–5 just to span the canonical goal manifold |
| Diffusion policy ([[diffusion-policy]], [[planning-as-diffusion]]) | Single-shot parameter generation instead of iterative denoising over action sequences; matches the open-loop one-trajectory constraint directly |

**External validation of the recipe.** DA-MMP (ICRA 2026) independently arrives at
sweep → compact trajectory parametrization → conditional flow matching conditioned on
goal + outcomes (90k planned throws; beats trained humans at ring-tossing); MMFP (RA-L 2025)
and DMMP validate the latent-manifold variant. **None of these has a wiki paper page yet —
run `/ingest` before citing them in a novelty statement.** DA-MMP's outcome-conditioning
trick (condition the flow on a handful of observed real outcomes) is the cheapest published
sim-to-real correction for exactly this pipeline, earmarked for the real-robot campaign.

## Evaluated by
- [[sim-stage-b-amortization-shootout]]
- [[sim-stage-d-gated-extensions]]
