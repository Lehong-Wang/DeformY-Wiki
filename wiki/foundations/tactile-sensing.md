---
title: "Tactile Sensing"
slug: "tactile-sensing"
domain: "Robotics"
status: mainstream
aliases: ["tactile sensors", "haptic sensing"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Tactile_sensor"
---

## Definition

A tactile sensor is a device that measures information arising from physical interaction with its environment. Tactile sensors are generally modeled after the biological sense of cutaneous touch which is capable of detecting stimuli resulting from mechanical stimulation, temperature, and pain. Tactile sensors are used in robotics, computer hardware and security systems. A common application of tactile sensors is in touchscreen devices on mobile phones and computing.

## Intuition (LLM analysis)

Vision sees what is uncovered; tactile feels what is in contact and gives high-bandwidth signals that detect slip, contact onset, and local geometry that no camera can resolve. For deformables and occluded contacts, tactile is often the only honest signal.

## Formal notation (LLM analysis)

Signal model varies by modality: optical-tactile (image of marker displacements + optional depth), capacitive (force per taxel), barometric (pressure per chamber), piezoresistive arrays, MEMS triaxial.

## Key variants (LLM analysis)

- Optical-tactile: GelSight, GelSlim, DIGIT, TacTip.
- Capacitive arrays (BioTac, e-skin).
- Barometric (Takktile).
- High-resolution magnetic-based (ReSkin).
- Distributed whole-arm skin.

## Known limitations (LLM analysis)

Signal is local and high-rate; data labeling is expensive. Sim-to-real for tactile is harder than for vision. Sensors are fragile and expensive at high resolution.

## Open problems (LLM analysis)

Tactile foundation models (cross-sensor, cross-task); cheap robust skin for whole-arm coverage; tactile-conditioned policies that exploit slip and contact-pressure cues.

## Relevance to active research (LLM analysis)

DLO manipulation — especially knot tying, suturing, and cable insertion — benefits enormously from tactile feedback; learned tactile policies are an active frontier in DLO research.
