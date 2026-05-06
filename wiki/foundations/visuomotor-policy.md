---
title: "Visuomotor Policy"
slug: "visuomotor-policy"
domain: "Robotics"
status: mainstream
aliases: ["pixel-to-action policy", "end-to-end visuomotor control"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

A visuomotor policy is an end-to-end mapping from raw visual observations (typically RGB or RGBD images) to low-level robot actions, learned jointly rather than via separate perception + control modules.

## Intuition (LLM analysis)

Decoupling perception (build a state estimate) and control (act on the state) loses signal at the interface; the perception module does not know what the controller needs. End-to-end visuomotor learning lets gradients shape the visual representation around the control task.

## Formal notation (LLM analysis)

Policy $\pi_\theta(a \mid o_t, \dots, o_{t-h})$ where $o$ is image (or image + proprioception). Trained by IL or RL with the visual encoder included in the backprop graph.

## Key variants (LLM analysis)

- CNN-MLP encoders (Levine et al. 2016, GPS-style).
- Transformer policies on patch / token features (RT-1, RT-2, OpenVLA).
- Diffusion-policy backbones with image / point-cloud conditioning.
- VLA models that share weights with vision-language pretraining.
- Latent-action policies (factor encoder out of the action head).

## Known limitations (LLM analysis)

Sample inefficient compared to state-based policies. Sensitive to camera placement and lighting. Sim-to-real for vision is harder than for proprioceptive control.

## Open problems (LLM analysis)

Generalization across embodiments and scenes; fusing 3D representations (NeRF, Gaussian Splatting, point clouds) with policies; calibration-free deployment.

## Relevance to active research (LLM analysis)

DLO manipulation is fundamentally a vision-driven task — the rope's shape can only be observed; visuomotor policies dominate this literature.
