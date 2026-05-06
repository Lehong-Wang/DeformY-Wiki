---
title: "Optimization"
slug: "optimization"
domain: "general"
status: mainstream
aliases: ["mathematical programming"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Mathematical_optimization"
---

## Definition

Mathematical optimization or mathematical programming is the selection of a best element, with regard to some criteria, from some set of available alternatives. It is generally divided into two subfields: discrete optimization and continuous optimization. Optimization problems arise in all quantitative disciplines from computer science and engineering to operations research and economics, and the development of solution methods has been of interest in mathematics for centuries.

## Intuition (LLM analysis)

Most ML and control problems pose: find parameters that make the system behave best, subject to constraints. Optimization is the toolbox — convex programs, gradient methods, sampling, search.

## Formal notation (LLM analysis)

$\min_{x \in \mathcal{X}} f(x) \;\; \mathrm{s.t.}\;\; g_i(x) \le 0,\; h_j(x) = 0$.

## Key variants

- Convex programs (LP, QP, SDP, conic).
- Nonlinear programming.
- Discrete / combinatorial optimization.
- Stochastic optimization.
- Bilevel / minimax (e.g. GANs).
- Bayesian optimization (sample-efficient global search).

## Known limitations (LLM analysis)

Non-convex landscapes admit only local guarantees. Constraints from real systems are often non-smooth (contacts, switching). Cost-function design is itself ill-posed.

## Open problems (LLM analysis)

Optimization on manifolds (SE(3), shape spaces); differentiable contact and physics; first-order methods that scale to trillion-parameter regimes with theoretical guarantees.

## Relevance to active research (LLM analysis)

Trajectory optimization, MPC, inverse kinematics, and RL all reduce to constrained optimization over different state/action/parameter spaces.
