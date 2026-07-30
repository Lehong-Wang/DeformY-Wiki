---
title: "Minimum-Jerk Trajectory"
slug: "minimum-jerk-trajectory"
domain: "Robotics"
status: mainstream
aliases: ["minimum jerk", "minimum-jerk model", "min-jerk trajectory"]
first_introduced: "Flash & Hogan 1985"
date_updated: "2026-06-16"
source_url: ""
---

## Definition (LLM analysis)

A minimum-jerk trajectory is the smooth point-to-point motion that minimizes the integral of squared jerk (the time derivative of acceleration) over the movement duration. Introduced by Flash and Hogan (1985) as a model of human reaching, it yields a closed-form quintic (fifth-order) polynomial in time and produces the characteristic smooth, bell-shaped velocity profile. It is the canonical compact representation of human-like, dynamically gentle motion and a common reference generator in robotics.

## Intuition (LLM analysis)

Jerk is how abruptly acceleration changes; high jerk means snappy, jarring motion that excites vibration and stresses actuators. Minimizing total squared jerk asks for the "gentlest possible" way to move from a start pose to an end pose in a given time. The optimal answer is fully determined by the boundary conditions (start/end position, velocity, acceleration) and produces a single smooth acceleration-then-deceleration with no wasted wiggling — which happens to match how humans naturally reach.

## Formal notation (LLM analysis)

For a one-dimensional move from $x_0$ to $x_f$ over duration $T$, minimize
$$J = \frac{1}{2}\int_0^T \left(\frac{d^3 x}{dt^3}\right)^2 dt.$$
The Euler-Lagrange solution is a fifth-order polynomial. With rest-to-rest boundary conditions ($\dot x = \ddot x = 0$ at both ends) and normalized time $\tau = t/T \in [0,1]$:
$$x(t) = x_0 + (x_f - x_0)\big(10\,\tau^3 - 15\,\tau^4 + 6\,\tau^5\big).$$
The velocity profile $\dot x(t) \propto \tau^2(1-\tau)^2$ is symmetric and bell-shaped, peaking at $\tau = 0.5$. The same form applies per-coordinate in task space.

## Key variants (LLM analysis)

- **Rest-to-rest quintic** — the standard closed form above (zero boundary velocity/acceleration).
- **Non-zero boundary conditions** — general quintic for through-motion with specified end velocity/acceleration.
- **Via-point minimum-jerk** — piecewise-smooth trajectories passing through intermediate points (Flash & Hogan extension).
- **Minimum-snap / higher-order** — minimize the 4th derivative (snap) for systems sensitive to even higher smoothness, e.g. quadrotors.
- **Minimum-torque-change** — a dynamics-based alternative criterion that depends on the arm's inertial model rather than pure kinematics.

## Known limitations (LLM analysis)

It is a purely kinematic criterion: it ignores actuator dynamics, torque limits, and contact, so the smooth reference may be dynamically infeasible or suboptimal for energy. It assumes a fixed duration $T$ chosen in advance, addresses unconstrained free-space motion (no obstacles), and is a descriptive model of unobstructed reaching rather than an optimal controller.

## Open problems (LLM analysis)

Reconciling kinematic smoothness with dynamic feasibility and contact constraints; principled duration selection; smooth blending of multiple minimum-jerk segments and via-points online; and embedding smooth point-to-point motion within rhythmic or reactive controllers.

## Relevance to active research (LLM analysis)

Minimum-jerk profiles are a standard, compact smooth-motion baseline and reference generator in manipulation and motion planning, and they connect to movement-primitive and limit-cycle controllers that aim to produce human-like discrete motions. Seeding it as a foundation lets ingested motion-generation papers link to a single definition of smooth point-to-point motion.
