---
title: "Learning dynamical systems with bifurcations"
authors: [Fares Khadivar, Ilaria Lauzana, Aude Billard]
venue: "Robotics and Autonomous Systems"
year: 2021
arxiv_id: null
doi: "10.1016/j.robot.2020.103700"
note_type: bibliography_only
sources: [field-research]
---

# LASA Hopf bifurcation: unified rhythmic + discrete DS control

**One-line gist**: A single time-invariant dynamical system embeds a Hopf bifurcation so the robot smoothly transitions between a stable limit cycle (rhythmic) and a point attractor (discrete) without switching controllers.

**Task/Method setup**: The DS is parameterized by a bifurcation parameter that continuously deforms the topology from periodic orbit to fixed-point; limit-cycle shape is learned from demonstration via diffeomorphism. Bifurcation parameters (amplitude, frequency, equilibrium location) are user-controllable. Validated on a real 7-DOF KUKA LWR 4+ arm (wiping task) and humanoid in simulation.

**Sim vs real**: Real robot experiments on KUKA LWR 4+; no sim-to-real gap discussion — system is model-free DS.

**Core idea / mechanism**:
- Hopf normal-form ODE governs the rhythmic phase; a single scalar bifurcation parameter drives the system from limit-cycle regime (r > 0) to stable-equilibrium regime (r < 0).
- The transition is smooth and continuous — no discrete switching logic, no jitter.
- Nonlinear limit-cycle shapes are encoded by learning a diffeomorphism from demonstrations and composing it with the Hopf normal form.
- Phase-space geometry: limit cycle lives in a 2-D plane; point attractor is co-located with the cycle's center, so convergence to goal is guaranteed after bifurcation.
- Code: github.com/epfl-lasa/SAHR_bifurcation

**Why it matters for OUR problem**:
- **Wind-up via Hopf**: directly implements the "wind-up into a stable limit cycle" module proposed in our approach — the limit cycle gives a repeatable, phase-coherent initial condition before the open-loop throw.
- **Compact action / smooth trajectory**: the DS produces smooth, continuously differentiable trajectories; compatible with our min-jerk / spline action decoder.
- **Transition to release**: the bifurcation mechanism provides a principled way to exit the limit cycle and converge to a throw-initiation posture, replacing ad-hoc switching logic.
- **No online adaptation needed**: the DS is time-invariant and feedforward once parameters are fixed — consistent with our open-loop execution constraint.
- **Calibration hook**: bifurcation parameters (amplitude, frequency) can be tuned during the one-time real calibration to match the physical rope's resonance without per-target adaptation.

**Key result**: Smooth rhythmic-to-discrete transitions on a real manipulator with no jitter; limit-cycle amplitude and frequency are independently controllable; shape learning via diffeomorphism generalizes to nonlinear orbits beyond the Hopf ellipse.
