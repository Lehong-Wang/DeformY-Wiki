---
title: "Discrete Elastic Rods"
slug: "discrete-elastic-rods"
domain: "Robotics"
status: mainstream
aliases: ["DER"]
first_introduced: "2008"
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Discrete Elastic Rods (DER) is a discrete differential geometry formulation of Kirchhoff/Cosserat rod dynamics introduced by Bergou et al. (2008/2010). It represents a rod as a polyline plus a scalar twist per segment and yields stable, fast, physically faithful simulation.

## Intuition (LLM analysis)

Rather than discretize a PDE, DER builds bending and twisting energies directly from discrete geometric quantities (curvature binormal, material frame increments). The result has small DOF count and runs at interactive rates, making it a workhorse in graphics and now robotics.

## Formal notation (LLM analysis)

Vertices $\{x_i\}$ define edges $e_i = x_{i+1}-x_i$. Discrete bending energy $E_b = \tfrac{1}{2}\sum_i \frac{1}{\bar\ell_i} \| (\kappa\mathbf{b})_i - \overline{(\kappa\mathbf{b})}_i \|^2$, twist energy $E_t = \tfrac{1}{2} \beta \sum_i (m_i - \bar m_i)^2 / \bar\ell_i$.

## Key variants (LLM analysis)

- Bergou-Audoly DER (graphics standard).
- Inextensible vs. stretchable formulations.
- DER with self-contact (capsule collisions).
- GPU / batched DER for parallel rollouts.
- Differentiable DER (DiffDER, gradient-friendly variants).

## Known limitations (LLM analysis)

Implicit time integration is required for stiff regimes. Self-contact handling adds significant cost. Plastic / inelastic effects need ad hoc extensions.

## Open problems (LLM analysis)

End-to-end differentiable DER with contact; learning-augmented DER for hysteresis; large-batch GPU DER simulators usable for policy training.

## Relevance to active research (LLM analysis)

Most modern DLO simulators in robotics (e.g. DiffSimDLO, DEF-Sim) use DER or close descendants; DER is the natural pairing for differentiable trajectory optimization on rope.
