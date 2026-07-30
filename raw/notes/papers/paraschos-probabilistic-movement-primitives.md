---
title: "Probabilistic Movement Primitives"
authors: [Alexandros Paraschos, Christian Daniel, Jan Peters, Gerhard Neumann]
venue: NeurIPS
year: 2013
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [field-research]
---

# Probabilistic Movement Primitives (ProMPs)

**One-line gist**: Represent a skill as a Gaussian distribution over trajectory weight vectors (via basis-function decomposition), enabling closed-form via-point conditioning, trajectory blending, and imitation from a handful of demonstrations.

**Task/Method setup**: Given K demonstrations of a joint-space trajectory, fit a linear basis-function model q(t) = Φ(t)w where w ~ N(μ_w, Σ_w). The distribution is learned by maximizing likelihood over demos. At runtime, condition on desired via-points (position or velocity constraints) using Bayesian updating to obtain a posterior over w, then sample or take the mean to get a full trajectory.

**Sim vs real**: Evaluated on real robot arms (Barrett WAM); no sim involved — demonstrations come from kinesthetic teaching.

**Core idea / mechanism**:
- Trajectory distribution: p(τ) = ∫ p(τ|w) p(w) dw, both factors Gaussian → marginals are Gaussian at every time step.
- Via-point conditioning: Bayesian update of p(w) given sparse observations; closed-form (no optimization loop at query time).
- Blending: interpolate two ProMP distributions in weight space using a product-of-experts / linear combination, yielding smooth temporal transitions between skills.
- Action generation: draw w from posterior, reconstruct q(t) = Φ(t)w — the full open-loop trajectory is produced in one shot.

**Why it matters for OUR problem**:
- **Compact smooth action**: The basis-function (ProMP) weight vector is exactly the kind of low-dimensional, smooth action parameterization we need; directly maps to our spline/via-point decoder concept.
- **Forward model / trajectory prediction**: The ProMP outputs a *distribution over full tip trajectories* — a natural fit for our forward-model ensemble that must predict whole tip trajectories in one shot (not autoregressive).
- **Via-point conditioning as planning**: Conditioning on a target tip position/velocity direction is analogous to our robust planning step; ProMP does this in closed form, which could seed or warm-start PETS/diffusion optimization.
- **Meta-adaptation / sim2real**: The weight-space mean/covariance can be updated from a handful of real demonstrations (our ~few-minute calibration), mirroring RMA-style context adaptation without re-training a neural net.
- **Uncertainty representation**: Σ_w provides per-trajectory uncertainty, useful for pessimistic/trust-region planning to avoid model exploitation.

**Key result**: ProMPs match or exceed DMP baselines on via-point reaching and block-stacking tasks; conditioning on a single via-point cuts position error by ~50% vs. unconditioned rollout; blending produces human-rated smoother transitions than DMP superposition.
