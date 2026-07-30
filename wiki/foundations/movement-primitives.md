---
title: "Movement Primitives"
slug: "movement-primitives"
domain: "Robotics"
status: mainstream
aliases: ["motor primitives", "DMP", "ProMP", "dynamic movement primitives", "probabilistic movement primitives"]
first_introduced: "Ijspeert, Nakanishi & Schaal 2002 (DMP); Paraschos et al. 2013 (ProMP)"
date_updated: "2026-06-16"
source_url: ""
---

## Definition (LLM analysis)

Movement primitives are compact, parameterized trajectory generators that encode reusable units of motion for robots. The two dominant families are Dynamical Movement Primitives (DMPs), which represent a movement as a stable attractor dynamical system modulated by a learnable nonlinear forcing term (supporting both discrete point-to-point and rhythmic motions), and Probabilistic Movement Primitives (ProMPs), which represent a distribution over trajectories and support via-point conditioning, blending, and combination through probabilistic operations.

## Intuition (LLM analysis)

Instead of specifying a motion point by point, a movement primitive captures its *shape* in a few parameters that can be relearned, retimed, and reused. DMPs do this by attaching a learnable "forcing" signal to a spring-like system that is guaranteed to settle at a goal: the spring ensures convergence and stability, while the forcing term sculpts the path on the way there. ProMPs instead model a whole *family* of demonstrated trajectories as a probability distribution, so you can condition on a desired via-point, blend two skills, or sample variations — all by manipulating Gaussians over the parameters.

## Formal notation (LLM analysis)

A discrete DMP augments a goal-attractor with a forcing term $f$ driven by a phase variable $s$ that decays monotonically (so $f \to 0$ and stability is preserved):
$$\tau \dot{v} = \alpha(\beta(g - x) - v) + f(s), \qquad \tau \dot{x} = v, \qquad \tau \dot{s} = -\alpha_s s,$$
where $f(s) = \frac{\sum_i \psi_i(s)\, w_i}{\sum_i \psi_i(s)}\, s\,(g - x_0)$ is a normalized weighted sum of basis functions $\psi_i$ with learnable weights $w_i$; a rhythmic DMP replaces the canonical system with a phase oscillator. A ProMP models a trajectory as $y_t = \Phi_t^\top w + \epsilon$ with $w \sim \mathcal{N}(\mu_w, \Sigma_w)$, so conditioning on a via-point is a Gaussian update on $w$ and blending is a product of Gaussians.

## Key variants (LLM analysis)

- **Discrete DMP** — point-to-point motion with a goal attractor and phase-gated forcing (reach, place).
- **Rhythmic DMP** — periodic motion driven by a phase oscillator (locomotion, wiping, stirring).
- **Probabilistic Movement Primitives (ProMP)** — distributions over trajectories with via-point conditioning, blending, and co-activation.
- **Kernelized / GMM primitives** — GMM/GMR (e.g. via TP-GMM) and kernelized movement primitives as alternative encodings.
- **Deep / neural movement primitives** — learned latent primitives and conditional neural movement primitives.

## Known limitations (LLM analysis)

DMPs are typically per-degree-of-freedom and can struggle to couple dimensions or generalize far from the demonstrated goal; their stability guarantee is purely kinematic and ignores contact/dynamics. ProMPs require multiple aligned demonstrations and a basis-function choice, and time-alignment (phase estimation) is fragile. Neither natively handles obstacles or hard constraints without augmentation.

## Open problems (LLM analysis)

Primitives that couple many degrees of freedom and generalize across goals and embodiments; tight integration with contact and force control; unifying discrete and rhythmic motion in one learnable system; and combining the compactness of primitives with the expressivity of modern generative trajectory models.

## Relevance to active research (LLM analysis)

Movement primitives are a foundational, parameter-efficient way to represent and learn manipulation and locomotion skills from demonstration, and they sit alongside minimum-jerk and limit-cycle controllers as compact motion generators. They connect to imitation-learning and rhythmic-motion work, so seeding this foundation lets ingested motion-skill papers wikilink here rather than redefining DMPs/ProMPs.
