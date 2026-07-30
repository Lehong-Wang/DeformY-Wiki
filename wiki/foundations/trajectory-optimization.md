---
title: "Trajectory Optimization"
slug: "trajectory-optimization"
domain: "Robotics"
status: mainstream
aliases: ["trajectory optimisation", "optimal trajectory generation"]
first_introduced: "Bryson & Ho 1969 (optimal control foundations)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Trajectory_optimization"
---

## Definition

Trajectory optimization is the process of designing a trajectory — a sequence of states and/or controls — that minimizes some measure of performance (a cost) while satisfying constraints, most importantly the system dynamics. It computes an open-loop solution to an optimal control problem and is used when a full closed-loop solution is unnecessary, impractical, or impossible. Executing only the first step of a freshly re-optimized trajectory at each timestep recovers Model Predictive Control.

## Intuition

Rather than reacting instant-by-instant, trajectory optimization plans the whole motion at once: "what sequence of actions, obeying the robot's dynamics and limits, gets from here to the goal at least cost?" Cost might encode goal-reaching, energy, smoothness, or obstacle avoidance; constraints encode the physics and actuator/joint bounds. The output is a feedforward plan, which is typically re-optimized as the world is re-observed (receding horizon) to regain feedback.

## Formal notation

A continuous-time problem seeks state $x(t)$ and control $u(t)$:
$$\min_{x(\cdot),\,u(\cdot)} \ \phi(x(T)) + \int_0^T \ell(x(t), u(t))\,dt \quad \text{s.t.}\quad \dot{x} = f(x, u),\ \ g(x, u) \le 0,\ \ x(0) = x_0.$$
Discretizing over a horizon $H$ gives a nonlinear program over $\{x_t, u_t\}_{t=0}^{H}$. **Shooting** methods parameterize controls and forward-simulate the dynamics (the states are implicit); **collocation** methods treat states and controls as decision variables and enforce dynamics as equality constraints at collocation points.

## Key variants

- **Shooting vs collocation** — simulate-the-dynamics (single/multiple shooting) vs dynamics-as-constraints (direct collocation).
- **Gradient-based (indirect/direct)** — DDP, iLQR, and NLP solvers (SQP, interior-point) that use derivatives of dynamics and cost.
- **Sampling-based / derivative-free** — CEM, MPPI, and random shooting that optimize by sampling action sequences and reweighting elites (the standard choice in model-based RL).
- **Receding-horizon execution (MPC)** — repeatedly solve and apply the first action to obtain feedback from an open-loop optimizer.

## Known limitations

The problem is generally non-convex, so solvers find local optima and are sensitive to initialization. Gradient-based methods require differentiable, well-conditioned dynamics; contact and deformation produce stiff, discontinuous gradients. Sampling-based methods avoid derivatives but scale poorly with horizon and action dimension. Real-time solution on hardware imposes tight compute budgets, and any model error directly corrupts the plan.

## Open problems (LLM analysis)

Robust real-time optimization through contact and deformation; global (not merely local) trajectory optimization; learned warm-starts and proposals to accelerate convergence; and principled handling of model uncertainty so optimized trajectories do not exploit model error.

## Relevance to active research (LLM analysis)

Trajectory optimization is the umbrella foundation for planning over actions and subsumes the sampling-based planners (CEM/MPPI) used in PETS and other model-based controllers, as well as the gradient-based iLQR/DDP lineage. It frames diffusion *planners* (Diffuser) as an alternative way to generate constraint-satisfying trajectories, connecting the planning-method papers being ingested.
