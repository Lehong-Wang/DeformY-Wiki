---
title: "Model Predictive Control"
slug: "model-predictive-control"
domain: "Robotics"
status: mainstream
aliases: ["MPC", "receding-horizon control"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Model_predictive_control"
---

## Definition

Model predictive control (MPC) is an advanced method of process control that is used to control a process while satisfying a set of constraints. Model predictive controllers rely on dynamic models of the process, most often linear empirical models obtained by system identification. The main advantage of MPC is the fact that it allows the current timeslot to be optimized, while keeping future timeslots in account. This is achieved by optimizing a finite time-horizon, but only implementing the current timeslot and then optimizing again, repeatedly, thus differing from a linear–quadratic regulator (LQR). Also MPC has the ability to anticipate future events and can take control actions accordingly. PID controllers do not have this predictive ability. MPC is nearly universally implemented as a digital control, although there is research into achieving faster response times with specially designed analog circuitry.

## Intuition (LLM analysis)

Solve a short-horizon optimization that respects current state and constraints; use the head of that plan, then roll forward and resolve. This handles disturbances and model error gracefully because the controller is constantly correcting itself.

## Formal notation (LLM analysis)

$\min_{u_{0:H-1}} \sum_{k=0}^{H-1} \ell(x_k, u_k) + \ell_f(x_H)$, subject to $x_{k+1} = f(x_k, u_k),\; g(x_k, u_k) \le 0$. Apply $u_0$, then re-solve at the next step.

## Key variants

- Linear MPC (QP solvers, real-time iteration).
- Nonlinear MPC (NMPC).
- Sampling-based MPC (CEM, MPPI).
- Tube / robust MPC.
- Learning-based MPC with neural-network or Gaussian-process dynamics.
- Differentiable MPC (treat the QP as a layer).

## Known limitations (LLM analysis)

Compute budget per step is tight. Performance depends on dynamics model accuracy. Handling discrete events (contacts, switching) is challenging.

## Open problems (LLM analysis)

MPC over learned latent dynamics for deformables; safe MPC with formal guarantees; warm-starting and amortizing the optimization to enable kHz-rate predictive control.

## Relevance to active research (LLM analysis)

Sampling-based MPC (MPPI) over learned DLO dynamics is a leading approach for cable / rope shaping; MPC is also the typical wrapper around model-based RL planners.
