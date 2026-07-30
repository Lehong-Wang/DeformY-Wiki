---
title: "Denoising Diffusion Probabilistic Models"
slug: "denoising-diffusion-probabilistic-models"
domain: "general"
status: mainstream
aliases: ["DDPM", "diffusion models", "score-based generative models", "denoising diffusion"]
first_introduced: "Sohl-Dickstein et al. 2015; Ho et al. 2020 (DDPM)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Diffusion_model"
---

## Definition

Denoising diffusion probabilistic models (DDPMs) are a class of latent-variable generative models that learn to reverse a gradual noising process in order to sample from a data distribution. A diffusion model has two components: a fixed forward process that incrementally corrupts data into Gaussian noise, and a learned reverse process that denoises step by step to generate new samples distributed like the training data. They are the substrate for diffusion policies and diffusion *planners*, and their sampling can be steered toward objectives via classifier or cost guidance.

## Intuition

Generation is framed as "learn to undo noise." During the forward process a datum is slowly destroyed by adding small amounts of Gaussian noise until it is indistinguishable from random noise — an easy distribution to sample. A neural network is trained to estimate, at each noise level, how to nudge a noisy sample back toward the data manifold. To generate, start from pure noise and apply this learned denoiser repeatedly. Because each step is small, the model only ever has to solve an easy local problem, which yields stable training and high sample quality. In planning, the "datum" is an entire trajectory, so denoising synthesizes a coherent plan from noise.

## Formal notation

The forward process adds noise over $T$ steps with schedule $\{\beta_t\}$: $q(x_t \mid x_{t-1}) = \mathcal{N}(x_t;\ \sqrt{1-\beta_t}\,x_{t-1},\ \beta_t I)$, with the closed form $x_t = \sqrt{\bar\alpha_t}\,x_0 + \sqrt{1-\bar\alpha_t}\,\epsilon$ where $\bar\alpha_t = \prod_{s\le t}(1-\beta_s)$ and $\epsilon \sim \mathcal{N}(0, I)$. A network $\epsilon_\theta(x_t, t)$ is trained with the simplified objective $\mathbb{E}_{x_0,\epsilon,t}\big[\|\epsilon - \epsilon_\theta(x_t, t)\|^2\big]$. Sampling runs the learned reverse process $p_\theta(x_{t-1}\mid x_t)$ from $x_T \sim \mathcal{N}(0, I)$. Guidance adds the gradient of an external objective, $\hat{\epsilon} = \epsilon_\theta - \sqrt{1-\bar\alpha_t}\,\nabla_{x_t}\log p(y\mid x_t)$ (classifier guidance) or a cost gradient, to steer samples.

## Key variants

- **DDPM** — the discrete-time denoising formulation (Ho et al. 2020).
- **Score-based / SDE models** — the continuous-time view via stochastic differential equations and score matching.
- **DDIM** — deterministic, non-Markovian sampling for far fewer steps.
- **Latent diffusion** — diffusion in a learned latent space for efficiency (e.g. Stable Diffusion).
- **Classifier / classifier-free / cost guidance** — conditioning and objective-steering mechanisms; the basis of diffusion planning.
- **Diffusion policies and planners** — diffusion over action sequences (policy) or full state-action trajectories (Diffuser).

## Known limitations

Sampling is slow: many sequential denoising steps make naive DDPMs costly for real-time control (mitigated by DDIM and distillation). Training requires substantial data and compute. Guidance can trade sample fidelity against constraint satisfaction and may push samples off-manifold. For planning, long trajectories and hard dynamic feasibility remain difficult.

## Open problems (LLM analysis)

Few-step or single-step sampling fast enough for closed-loop robot control; guidance that guarantees dynamic feasibility and hard-constraint satisfaction; scaling diffusion planners to long horizons and high-dimensional deformable states; and principled composition of multiple cost/guidance signals.

## Relevance to active research (LLM analysis)

DDPMs are the generative engine beneath the Diffuser planner being ingested and beneath diffusion policies for manipulation; framing planning as guided trajectory denoising is an alternative to sampling-based trajectory optimization. Seeding this foundation lets the ingested Diffuser page link here instead of re-deriving the forward/reverse diffusion mechanics.
