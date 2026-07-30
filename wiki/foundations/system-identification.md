---
title: "System Identification"
slug: "system-identification"
domain: "Robotics"
status: mainstream
aliases: ["sysid", "system ID", "model identification", "parameter estimation"]
first_introduced: "Ho & Kalman 1965; Ljung 1987 (textbook)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/System_identification"
---

## Definition

System identification is the field that uses statistical methods to build mathematical models of dynamical systems from measured input-output data. It also covers the optimal design of experiments for generating informative data and the reduction of resulting models. A common strategy starts from measurements of a system's behavior and external inputs and fits a mathematical relationship between them without modeling the internal mechanism in detail (black-box identification); gray-box and white-box variants incorporate increasing amounts of physical structure. It is the foundation for simulator calibration and adaptive control.

## Intuition

If you do not have equations for a system but you can poke it and watch how it responds, system identification turns those input-output traces into a predictive model. You excite the system (commanded torques, reference signals), record the responses (positions, forces), and fit parameters — masses, frictions, time constants — so the model reproduces the data and, crucially, predicts responses to new inputs. The art is exciting the system enough (persistent excitation) that the data actually pins the parameters down.

## Formal notation

Given input-output data $\{(u_t, y_t)\}_{t=1}^{N}$ and a parameterized model $\hat{y}_t = g(u_{1:t};\, \theta)$, identification solves the prediction-error problem
$$\hat\theta = \arg\min_\theta \ \sum_{t=1}^{N} \big\| y_t - \hat{y}_t(\theta) \big\|^2 \quad \big(\text{or the maximum-likelihood/Bayesian analogue}\big).$$
Classical linear families include ARX/ARMAX, output-error, and state-space (subspace) models; consistency requires the input to be *persistently exciting*. Nonlinear and physics-based identification fit ODE/parameter sets (e.g. inertial parameters of a manipulator) and increasingly use neural networks for the residual or full dynamics.

## Key variants

- **Black-box vs gray-box vs white-box** — no physical structure, partial structure, or full first-principles structure with unknown parameters.
- **Linear model families** — ARX, ARMAX, output-error, Box-Jenkins, and subspace state-space identification.
- **Nonlinear / physics-based** — fitting inertial/friction parameters of rigid-body dynamics, or NARX/neural dynamics models.
- **Online / recursive identification** — recursive least squares and Kalman-filter-based estimation for adaptive control.
- **Online domain/parameter adaptation** — RL-era analogues that infer changing physical parameters at deployment (e.g. RMA's latent factors, Learning-to-Adapt's online model update).

## Known limitations

Identifiability fails without sufficiently exciting inputs — some parameters simply cannot be recovered from the available data. Results are sensitive to noise, unmodeled dynamics, and model-structure choice (bias-variance). Nonlinear identification can be non-convex with many local optima, and offline-identified models drift as the real system changes (wear, payload), motivating online/adaptive variants.

## Open problems (LLM analysis)

Data-efficient identification of contact-rich and deformable-object dynamics; principled uncertainty quantification over identified parameters; safe online identification under closed-loop control; and hybrid models that combine first-principles structure with learned residuals without overfitting.

## Relevance to active research (LLM analysis)

System identification is the classical ancestor of modern online adaptation: calibrating a simulator (closing the reality gap) and inferring shifting physical parameters at runtime are exactly what RMA and Learning-to-Adapt do with learned latents and fast model updates. Seeding it as a foundation grounds those ingested method papers in the estimation theory they build on.
