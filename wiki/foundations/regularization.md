---
title: "Regularization"
slug: "regularization"
domain: "general"
status: mainstream
aliases: ["weight decay", "L1", "L2"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Regularization_(mathematics)"
---

## Definition

In mathematics, statistics, finance, and computer science, particularly in machine learning and inverse problems, regularization is a process that converts the answer to a problem to a simpler one. It is often used in solving ill-posed problems or to prevent overfitting.

## Intuition (LLM analysis)

Unrestricted models memorize training data. Regularization shrinks the effective hypothesis class — by adding a penalty to the loss, holding out parts of the network, stopping training early, or augmenting the data — so the learner must explain the data with a simpler function.

## Formal notation (LLM analysis)

Penalty form: $L_{\mathrm{reg}}(\theta) = L(\theta) + \lambda R(\theta)$, where common $R$ are $\|\theta\|_2^2$ (Tikhonov / weight decay) or $\|\theta\|_1$ (lasso / sparsity).

## Key variants

- Tikhonov / L2 / weight decay.
- L1 / lasso (sparsity).
- Elastic net (L1+L2).
- Dropout (random unit masking).
- Early stopping.
- Data augmentation (regularization through invariances).
- Spectral norm and Lipschitz constraints.

## Known limitations (LLM analysis)

Penalty strength must be tuned. Some forms (dropout, BN) interact non-trivially with batch statistics. On large pretrained models, naive weight decay can hurt rather than help.

## Open problems (LLM analysis)

Implicit regularization of optimizers vs. explicit penalties, regularization for distribution shift, and theoretically grounded hyperparameter selection.

## Relevance to active research (LLM analysis)

Robot-learning policies trained on small demonstration datasets are highly susceptible to overfitting; regularization choices materially affect sim-to-real and held-out performance.
