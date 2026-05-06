---
title: "Visual Servoing"
slug: "visual-servoing"
domain: "Robotics"
status: mainstream
aliases: ["VS", "image-based visual servoing", "position-based visual servoing"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Visual_servoing"
---

## Definition

Visual servoing, also known as vision-based robot control and abbreviated VS, is a technique which uses feedback information extracted from a vision sensor to control the motion of a robot. One of the earliest papers that talks about visual servoing was from the SRI International Labs in 1979.

## Intuition (LLM analysis)

If you can see what 'done' looks like in the camera, control the robot until the image matches. IBVS works directly on pixel features and avoids 3D reconstruction; PBVS computes a pose error in 3D.

## Formal notation (LLM analysis)

IBVS: $\dot s = L_s\,v_c$, with $s$ image features, $L_s$ the interaction matrix, and $v_c$ camera twist; control law $v_c = -\lambda L_s^+ (s - s^*)$.

## Key variants

- Image-based (IBVS).
- Position-based (PBVS).
- Hybrid 2.5D.
- Deep visual servoing (learned features and/or interaction matrices).
- Shape servoing (visual servoing on deformable-object shape descriptors).

## Known limitations (LLM analysis)

Local convergence; requires features that remain in view. Calibration sensitive. Camera-Jacobian estimation is fragile near depth ambiguities.

## Open problems (LLM analysis)

Visual servoing on learned latent descriptors (NeRF / 3DGS features); end-to-end visuomotor policies that subsume servoing; servoing for soft / continuum robots.

## Relevance to active research (LLM analysis)

Shape servoing — closed-loop control on a DLO's silhouette or keypoint set — is a direct descendant of visual servoing and remains a strong baseline for DLO shape control.
