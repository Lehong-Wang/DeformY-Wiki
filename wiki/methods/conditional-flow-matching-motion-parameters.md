---
name: "Conditional Flow Matching over Motion-Primitive Parameters"
slug: conditional-flow-matching-motion-parameters
type: architecture
tags: [flow-matching, conditional-generative-model, movement-primitives, amortized-inference, one-to-many-inverse, goal-conditioned, trajectory-generation, robot-learning]
source_papers: []
parent_methods: []
child_methods: []
realizes_concepts:
- "[[execution-outcome-conditioned-trajectory-generation]]"
- "[[multimodal-action-distributions-behavior-cloning]]"
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

> **Terminology warning (added 2026-07-30, from reading the primary sources).** This page's
> name collides two different senses of "conditional". In *Conditional Flow Matching*,
> "conditional" means the probability path is conditioned on a **data sample** $x_1$ (or a
> latent $z$) solely to make the loss tractable — Lipman et al.'s paper is *unconditional*
> image synthesis throughout. Conditioning on a **goal** is a separate mechanism the
> literature calls **guidance** / conditional generation, implemented by handing the network
> an extra input $y$. Both senses are in play here and they are independent.
> **Practical consequence: `torchcfm`'s `ConditionalFlowMatcher` does *not* do goal
> conditioning** — it only supplies $(x_t, u_t)$ training targets; passing $g$ into the
> network is the caller's job. Precise description of this method: *a goal-guided velocity
> field trained with the CFM loss.* See [[flow-matching]].

A goal-guided flow-matching network models p(w | g): it represents the solution **set**
rather than a point estimate. Three properties matter here, in order of importance:

1. **Multi-candidate sampling is free** — exactly what
   [[sim-verified-best-of-n-selection]] consumes. A regressor gives one candidate; the
   verifier needs N.
2. **Fixing the noise seed while sweeping the goal yields continuous goal→motion families.**
   This is the "target-space continuity" property previously hoped for from an MMP++ latent —
   obtained here without building an autoencoder.

   **Verified as a mathematical claim (2026-07-30):** sampling integrates the ODE
   $\dot x = u^\theta_t(x \mid g)$ from a fixed $X_0$, and ODE solutions depend continuously on
   parameters, so $g \mapsto w(g)$ is continuous for a trained network. **The caveat that must
   travel with it:** continuity is not validity. If $p(w\mid g)$ is multimodal and mode
   dominance switches as $g$ sweeps, the path moves smoothly but can traverse low-density
   regions — i.e. invalid swings between families. The deployment verifier is what closes that
   gap, so this property does not remove the need for one.

   **Separately, this claim was challenged on 2026-07-30 and needs its escape stated, not
   assumed.**
   [[da-mmp-learning-coordinated-accurate-throwing]]'s autoencoder ablation found that flow
   matching over *raw* trajectory parameters produces *"irregular and oscillatory"* and
   *"rarely executable"* joint profiles — i.e. the manifold was load-bearing for
   **executability**, not merely for statistical smoothness. The escape here is real but
   specific: this method's decoder ([[smooth-basis-swing-parameterization]]) enforces
   C²-smoothness and joint/velocity/acceleration limits **by construction for every
   parameter vector, including out-of-distribution ones**, via duration auto-scaling — so
   there is no such thing as an inexecutable sample to begin with. DA-MMP's raw parameters
   carried no such guarantee. If that decoder property ever weakens, this argument fails
   with it and the autoencoder stage comes back on the table.
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

Implementation stack: `torchcfm` (`atong01/conditional-flow-matching`, **MIT**, pip-installable,
~2.6k stars — verified 2026-07-30) + `ALRhub/MP_PyTorch`, with
`ScheiklP/movement-primitive-diffusion` as the architectural reference. **No end-to-end base
repo exists** — DA-MMP, DMMP and MMFP have all released no code (re-checked 2026-07-30).
Visuomotor policy frameworks should be avoided; their abstractions all mismatch one-shot
parameter generation.

**Two construction choices, settled from the primary sources** (see [[flow-matching]]):

- **Use the I-CFM / linear form**, not the Gaussian-path form: interpolate
  $X_t = tX_1 + (1-t)X_0$ and regress onto the constant velocity $X_1 - X_0$. Simpler, admits
  a non-Gaussian source, and it is where the field converged — Rectified Flow is the $\sigma=0$
  case of exactly this, and Stochastic Interpolants a reparameterization.
- **Use OT-CFM coupling** (`ExactOptimalTransportConditionalFlowMatcher`): sampling
  $(x_0, x_1)$ from a minibatch optimal-transport plan rather than independently costs **<1%
  training overhead** and yields straighter flows, i.e. fewer ODE steps per sample. That
  compounds here, because [[sim-verified-best-of-n-selection]] multiplies sampling cost by N —
  at N = 64 the NFE saving is the difference between a comfortable and an awkward deployment
  budget. Tong et al. find OT-CFM's advantage concentrated exactly in the low-NFE regime.

## Assumptions

- The hindsight pool covers the goal regions where claims will be made. Coverage is measured
  by [[direction-reachability-atlas]], not assumed.
- **Guidance works at all in this regime.** Unstated until now, and it is not free:
  [[flow-matching]]'s own reference guide notes guidance is most effective when *many* targets
  share one conditioning value, and "more challenging" when the conditioning variable is
  complex and non-repeating. Per-timestep relabeling gives the inverted structure — **one w
  paired with ~10² distinct goals**, so almost no two training pairs share a g. CVT-balancing
  over goal cells and the network's smoothness in g supply the sharing implicitly; that should
  be verified with a diagnostic rather than assumed. It also predicts a **goal-embedding
  robustness regularizer** would help — independently the largest single ablation effect in
  [[motion-manifold-flow-primitives-task-conditioned]].
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

**External validation of the recipe (all three ingested and verified 2026-07-30).**
[[da-mmp-learning-coordinated-accurate-throwing]] (Chu & Xu, **arXiv 2025 — an independent
group, not the Lee line, and *not* ICRA 2026 as earlier notes claimed**) independently
arrives at sweep → compact trajectory parametrization → conditional flow matching
conditioned on goal + outcomes: 90k planner-generated throws, 64-D latent, conditioned on
only 60 executed real throws, 60% real ring-toss vs 56.7% for trained human experts.
[[motion-manifold-flow-primitives-task-conditioned]] (RA-L 2025) and
[[differentiable-motion-manifold-primitives-reactive-motion]] (ICRA 2026) validate the
latent-manifold variant. None has released code.

Three qualifications that matter for how strongly this recipe can be cited:

- **DA-MMP labels one outcome per trajectory** — 60 throws, 60 training pairs, conditioning
  on the single executed landing point via classifier-free guidance. Verified from the text.
  This is what leaves the per-timestep composition in
  [[per-timestep-hindsight-relabeling]] unclaimed.
- **MMFP conditions on discrete task labels** (Sentence-BERT embeddings of ~15 finite tasks),
  evaluated by a self-trained classifier and MMD — never a metric error. It is a partial
  precedent only; DA-MMP is the continuous-goal member of that line.
- **DA-MMP executes one sample with no verifier**, and eats a 40% failure rate — the
  strongest available circumstantial argument for
  [[sim-verified-best-of-n-selection]].

DA-MMP's outcome-conditioning trick (condition the flow on a handful of observed real
outcomes) remains the cheapest published sim-to-real correction for exactly this pipeline,
earmarked for the real-robot campaign.

**Candidate additional arm — TMO.** [[trajectory-manifold-optimization]] (from DMMP) is
architecture-agnostic: fine-tune the conditional generator against a differentiable task
objective plus a squared-ReLU constraint penalty, with the conditioning variable sampled
uniformly over the full continuous goal space. Because $q(t;w)$ is analytic in this method's
basis, it drops onto this head directly as a **B3-TMO arm** — cheaper and better-motivated
than B4. Its reported effect is large where it matters: DMMP's seen→unseen generalization
holds 95.8 → 94.1 with TMO, against MMFP's 77.4 → 15.0 collapse without it. Caveat: only
*arm-side* constraints are TMO-able — the rope's tip is not a closed-form differentiable
function of $w$, since the simulator sits in between.

## Evaluated by
- [[sim-stage-b-amortization-shootout]]
- [[sim-stage-d-gated-extensions]]
