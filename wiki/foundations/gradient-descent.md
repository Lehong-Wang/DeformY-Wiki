---
title: "Gradient Descent"
slug: "gradient-descent"
domain: "general"
status: mainstream
aliases: ["steepest descent"]
first_introduced: "1847"
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Gradient_descent"
---

## Definition

Gradient descent is a method for unconstrained mathematical optimization. It is a first-order iterative algorithm for minimizing a differentiable multivariate function.

## Intuition (LLM analysis)

At every step you stand on the loss surface, look around for the steepest downhill direction, and take a small step that way. Over many small steps you converge toward a local minimum. The step size (learning rate) controls speed vs. stability.

## Formal notation (LLM analysis)

$\theta_{t+1} = \theta_t - \eta \nabla_\theta L(\theta_t)$, with learning rate $\eta>0$. Stochastic variants replace the full gradient with a mini-batch estimate.

## Key variants

- Batch GD: full-dataset gradient.
- Stochastic GD (SGD): one or a mini-batch of examples per step.
- Momentum / Nesterov: smooths past gradients.
- Adam / RMSProp / AdaGrad: per-parameter adaptive learning rates.
- Natural / preconditioned gradient: rescales by curvature (e.g. Fisher information).

## Known limitations (LLM analysis)

Step-size sensitive. Stalls in saddles / flat regions in high-dimensional non-convex landscapes. Pure GD is slow on ill-conditioned problems unless preconditioned.

## Open problems (LLM analysis)

How does implicit bias of stochastic mini-batch dynamics shape generalization in over-parameterized models? Why do simple optimizers like SGD outperform second-order methods on deep nets despite worse local guarantees?

## Relevance to active research (LLM analysis)

Every gradient-based learning system in this wiki — visuomotor policies, diffusion policies, dynamics models — ultimately reduces to a gradient-descent variant under the hood.
