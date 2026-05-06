---
title: "Shape Servoing"
slug: "shape-servoing"
domain: "Robotics"
status: mainstream
aliases: ["DLO shape control"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Shape servoing is closed-loop control that drives a deformable object's shape toward a target by estimating, online, a deformation Jacobian that maps small robot motions to small shape changes, and inverting it inside a feedback controller.

## Intuition (LLM analysis)

DLOs are too high-dimensional and uncertain to model exactly. Shape servoing sidesteps the model: perturb a little, measure how the rope's shape descriptor changed, fit a local linear map, then take the descent step that reduces shape error. Repeat.

## Formal notation (LLM analysis)

Shape descriptor $s \in \mathbb{R}^d$ (Fourier coefficients, keypoints, latent code). Local linear model: $\dot s = J_d(q, s)\,\dot q$. Control: $\dot q = J_d^+ \,K(s^* - s)$. Online estimator: e.g. Broyden-style update of $\hat J_d$ from observed pairs $(\Delta q, \Delta s)$.

## Key variants (LLM analysis)

- Fourier-feature shape servoing.
- Keypoint / catenary shape servoing.
- Latent shape servoing on autoencoder embeddings.
- Learning a global deformation model with online correction.
- Bimanual / multi-robot shape servoing.

## Known limitations (LLM analysis)

Local linearization fails at large shape errors and topology changes (knots forming). Online Jacobian estimation is noisy. Cannot plan around obstacles without an outer planner.

## Open problems (LLM analysis)

Provably stable global shape servoing with topology awareness; combining shape servoing with diffusion-policy priors for warm-starting; tactile-augmented shape servoing.

## Relevance to active research (LLM analysis)

Shape servoing remains a strong baseline and a key building block for DLO manipulation; modern learned approaches typically inherit its descriptor-and-Jacobian framing.
