---
title: "Impedance Control"
slug: "impedance-control"
domain: "Robotics"
status: mainstream
aliases: ["compliance control", "admittance control"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Impedance_control"
---

## Definition

Impedance control is an approach to dynamic control relating force and position. It is often used in applications where a manipulator interacts with its environment and the force position relation is of concern. Examples of such applications include humans interacting with robots, where the force produced by the human relates to how fast the robot should move/stop. Simpler control methods, such as position control or torque control, perform poorly when the manipulator experiences contacts. Thus impedance control is commonly used in these settings.

## Intuition (LLM analysis)

Stiff position control fights against the world; impedance control behaves like an instrumented spring, yielding when something pushes back. This makes contact-rich and deformable manipulation safer and more robust.

## Formal notation (LLM analysis)

$F_{ext} = M_d \ddot{\tilde x} + B_d \dot{\tilde x} + K_d \tilde x$, with $\tilde x = x - x_d$. Implement as torque law $\tau = J^\top(M_d \ddot{\tilde x} + B_d \dot{\tilde x} + K_d \tilde x) + \tau_{\mathrm{dyn comp}}$.

## Key variants

- Cartesian impedance (task-space).
- Joint impedance.
- Variable impedance (tune K, B online).
- Admittance control (force-sensed, position-commanded inverse).
- Learned variable impedance (RL or LfD).

## Known limitations (LLM analysis)

Stability depends on environment stiffness; must avoid algebraic loops. Requires accurate dynamics compensation or a torque-controlled robot. Variable impedance must guarantee passivity.

## Open problems (LLM analysis)

Learning task-conditional variable impedance from demonstration; passivity-preserving learned impedance; impedance for soft / deformable end-effectors.

## Relevance to active research (LLM analysis)

Manipulating DLOs against fixtures, knots, or human partners benefits enormously from impedance control; it is the standard underlying low-level controller for DLO learning research.
