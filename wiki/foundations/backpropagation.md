---
title: "Backpropagation"
slug: "backpropagation"
domain: "general"
status: mainstream
aliases: ["backprop", "reverse-mode automatic differentiation"]
first_introduced: "1986"
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Backpropagation"
---

## Definition

In machine learning, backpropagation is a gradient computation method commonly used for training a neural network in computing parameter updates.

## Intuition (LLM analysis)

Forward pass: feed inputs through layers and record activations. Backward pass: starting at the loss, send error signals backward through each layer, multiplying local Jacobians; each parameter receives the gradient that says how much nudging it would change the loss.

## Formal notation (LLM analysis)

For a composition $f = f_L \circ \dots \circ f_1$, $\partial L / \partial \theta_\ell$ factors via the chain rule: $\partial L/\partial \theta_\ell = (\partial L/\partial h_L)\,J_L \dots J_{\ell+1}\,(\partial f_\ell/\partial \theta_\ell)$.

## Key variants

- Reverse-mode automatic differentiation (general case).
- Backprop through time (BPTT) for RNNs.
- Truncated BPTT, gradient checkpointing for memory.
- Implicit / equilibrium backprop for fixed-point models.

## Known limitations (LLM analysis)

Requires storing intermediate activations (memory cost is depth-linear). Can suffer from vanishing/exploding gradients without normalization or skip connections. Not biologically plausible.

## Open problems (LLM analysis)

Forward-only credit assignment, local learning rules, alternatives that match GPU efficiency without the full backward graph (e.g., feedback alignment, predictive coding).

## Relevance to active research (LLM analysis)

Backprop is the universal differentiation engine behind every learned policy, dynamics model, and perception network this wiki tracks.
