---
title: "Diffusion Policy"
slug: "diffusion-policy"
domain: "Robotics"
status: mainstream
aliases: ["DDPM policy", "conditional diffusion policy"]
first_introduced: "2023"
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Diffusion Policy is a class of visuomotor policies that models the conditional distribution of robot actions given an observation as a denoising diffusion process, enabling expressive multi-modal action prediction. Introduced by Chi et al. (2023).

## Intuition (LLM analysis)

Manipulation often has multiple valid actions for the same observation (which way to grasp, which side to push). Regression-based BC averages these and produces incoherent actions. Diffusion samples from the action distribution rather than predicting a mean, so it captures multi-modality and produces smooth, temporally consistent action sequences.

## Formal notation (LLM analysis)

Train a score / noise-prediction network $\epsilon_\theta(a^k, k, o)$ to denoise an action chunk $a^0$ at noise level $k$, conditioned on observation $o$. Sample by iterative DDPM/DDIM from $a^K \sim \mathcal{N}(0,I)$ back to $a^0$. Action chunking (predict $T_a$ steps, execute $T_o$) provides temporal smoothness.

## Key variants (LLM analysis)

- CNN-based U-Net diffusion policy.
- Transformer-conditioned diffusion (DiT-style).
- 3D Diffusion Policy (point-cloud conditioning).
- Equivariant Diffusion Policy.
- Flow-matching variants (faster sampling).
- Consistency / one-step distillation for real-time control.

## Known limitations (LLM analysis)

Inference cost (multi-step sampling) limits closed-loop frequency. Conditioning on raw images alone is brittle to viewpoint and lighting shifts. Requires sizeable demonstration datasets to be competitive.

## Open problems (LLM analysis)

Single-step or few-step distillation that preserves multi-modality; principled goal/language conditioning; uncertainty estimation for safe deployment; integration with model-based planning.

## Relevance to active research (LLM analysis)

Diffusion Policy and its variants are the dominant policy class for current DLO manipulation research and underpin open-source generalist policies like Octo and π0.
