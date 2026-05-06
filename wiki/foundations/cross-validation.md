---
title: "Cross-Validation"
slug: "cross-validation"
domain: "general"
status: mainstream
aliases: ["k-fold", "CV"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Cross-validation_(statistics)"
---

## Definition

Cross-validation, sometimes called rotation estimation or out-of-sample testing, is any of various similar model validation techniques for assessing how the results of a statistical analysis will generalize to an independent data set.
Cross-validation includes resampling and sample splitting methods that use different portions of the data to test and train a model on different iterations. It is often used in settings where the goal is prediction, and one wants to estimate how accurately a predictive model will perform in practice. It can also be used to assess the quality of a fitted model and the stability of its parameters.

## Intuition (LLM analysis)

A single train/val split gives a noisy estimate of out-of-sample error. Splitting many ways and averaging cancels noise and reveals model variance across data subsets.

## Formal notation (LLM analysis)

k-fold CV: partition data into k folds, train on $k-1$ and validate on the remaining one, rotating; report mean and variance of the validation metric.

## Key variants

- k-fold (typically k=5 or 10).
- Leave-one-out (k=N).
- Stratified (preserve class ratios).
- Group / leave-one-subject-out (preserve grouping like robot, scene, demonstrator).
- Time-series / nested CV (respect temporal causality).

## Known limitations (LLM analysis)

Expensive on large models. Standard k-fold breaks under temporal or hierarchical structure. Information leakage if preprocessing fits on the entire dataset before splitting.

## Open problems (LLM analysis)

Cheap CV proxies for foundation-scale models; principled CV under heavy distribution shift.

## Relevance to active research (LLM analysis)

When evaluating learned manipulation policies, leave-one-trajectory-out and leave-one-object-out CV are far more honest than random splits, which often leak nearby states.
