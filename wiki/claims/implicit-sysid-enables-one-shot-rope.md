---
title: Implicit sysID encoder + goal-conditioned action predictor enables one-shot real-deployment for rope-as-tool 3D-target transport of rigid payloads
slug: implicit-sysid-enables-one-shot-rope
status: weakly_supported
confidence: 0.65
tags:
- implicit-system-identification
- rope-manipulation
- dynamic-manipulation
- one-shot
- sim-to-real
- heterogeneous-system
- soft-tool-use
domain: Robotics
source_papers:
- '[[implicit-physics-aware-policy-dynamic-manipulation]]'
evidence:
- source: implicit-physics-aware-policy-dynamic-manipulation
  type: supports
  strength: moderate
  detail: ICRA 2025. IPA's two-stage architecture (predefined SysID probe → ResNet that consumes (probe, response, depth, segmentation, aux) → cruising-velocity regression) reaches 72.5% SR on 50 simulated O2T cases (vs. 4.4% RND, 20.0% iterative SQ-RND, 16.7% IPA w/o SysID) and 62.5% SR on 8 real cases without any real-world training, while iterative residual baselines and explicit-physics methods do not transfer one-shot to varying friction. Sim-to-real Vel-Diff degrades only from 0.163 to 0.151 — i.e. the implicit-physics encoding survives the friction shift induced by adding a baking tray to the real setup.
conditions: "Single-paper preprint result with limited real-world evaluation (n=8). \nThe claim is established only for a single moving primitive (base-joint rotation with trapezoidal velocity profile), a single payload class (boxes), a single rope class, and a fixed predefined SysID probe action. Generalisation to other moving primitives, multi-stage casts, learned probes, or non-box payloads is not demonstrated. SR figures bake in the 0.5 m position-tolerance success threshold, which is task-specific.\n"
date_proposed: 2026-05-06
date_updated: 2026-05-06
---
## Statement

A two-stage learned policy that (i) executes a fixed predefined high-acceleration probing action on a heterogeneous rope+payload system, encodes the resulting trajectory map as an implicit physics representation, and (ii) consumes that representation alongside a depth + segmentation observation and a goal location to regress a single cruising velocity, can — when trained only in domain-randomised simulation — transport a rigid payload to a distant occluded 3D target on the *first* real-world attempt with success rate substantially above iterative-feedback and explicit-physics baselines.

## Evidence summary

The principal evidence is the IPA paper's headline simulated and real-world success rates (see `evidence`). Three internal comparisons strengthen the claim:

- The drop from IPA (72.5%) to IPA-w/o-SysID (16.7%) isolates the implicit-physics encoder as the key lever — not the ResNet, not the action parametrisation, not the depth+segmentation observation.
- Adding a configuration prediction critic (CPN) on top of IPA *hurts* (40.5% < 72.5%), suggesting the implicit-sysID-then-commit pattern is not strictly improved by adding an inner refinement loop in this regime.
- Real-world performance generalises from sim with no fine-tuning across two heterogeneous-system instances and a friction shift induced by placing the payload on a baking tray.

## Conditions and scope

- **Task class**: one-shot goal-conditioned dynamic transport of a rigid payload via a soft tool with a single trapezoidal velocity profile.
- **Payload class**: rigid boxes, sized and weighted within the training distribution.
- **Tool class**: ropes (DLOs), with length / radius / mass within the training distribution.
- **SysID probe**: predefined fixed $\bar{a} = (3.14, 1.04, 3.14, 8)$ — *not* learned, so the result depends on this probe exciting the relevant physics modes.
- **Environment**: stationary obstacles; UR5e + Robotiq 2F-85 hardware (or equivalent).

Outside these conditions — e.g. multi-stage casting, free-end cables without payload, novel moving primitives, learned probes — the claim is unsupported.

## Counter-evidence

- Real-world n=8 is small; a 62.5% SR has wide confidence bounds.
- The Configuration-Prediction-Network result (IPA + CPN underperforms IPA) cuts both ways — it strengthens the implicit-sysID-then-commit framing in this specific regime, but suggests the claim might *not* extend cleanly to settings where forward-model critics matter (e.g. deformable shape control rather than rigid-target transport).
- No external replication has been published yet; the IPA paper is a single-paper preprint at the time of this entry.
- The SysID probe is fixed and hand-designed; in environments where it fails to excite friction or rope-mass differences (e.g. extreme stiffness or mass), the implicit encoding may degrade silently to the no-SysID baseline.

## Linked ideas

(none yet)

## Open questions

- Does the claim extend to **learned probes**? A jointly trained $\bar{a}$ that maximises mutual information with the latent physics could either tighten or break the result.
- Does it extend to **multi-stage casts** where the policy must emit a sequence of velocity profiles and the SysID encoding must remain valid across the sequence?
- Does the implicit encoding **transfer across tasks**? E.g. take $\bar{\tau}$ from object transport and reuse it in rope whipping or DLO shape control without retraining the head.
- Why does CPN hurt in this regime, and is there a *predictive* variant that helps?
- What is the **sample-efficiency curve** of the SysID probe — how does SR scale with $\bar{\tau}$ resolution and probe horizon length?
