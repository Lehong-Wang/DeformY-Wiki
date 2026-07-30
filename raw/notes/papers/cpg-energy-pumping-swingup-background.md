---
title: "Background: CPG / Oscillator Control and Energy-Pumping Swing-Up"
authors:
  - Spong, M.W.
  - Nakanishi, J.
  - Fukuda, T.
  - Koditschek, D.E.
venue: "IEEE Control Systems Magazine; IEEE Trans. Robotics and Automation"
year: 1995
arxiv_id: null
doi: "[UNCONFIRMED]"
note_type: bibliography_only
sources: [field-research]
key_refs:
  - "Spong 1995 — The swing-up control problem for the Acrobot. IEEE Control Systems Mag. 15(1):49–55"
  - "Nakanishi, Fukuda, Koditschek 2000 — A brachiating robot controller. IEEE Trans. Robot. Autom. 16(2):109–123"
---

# CPG / Oscillator Control and Energy-Pumping Swing-Up

**One-line gist**: Energy-based pumping injects mechanical energy into an underactuated chain until a target limit cycle is reached; CPGs encode that limit cycle as an autonomous oscillator whose steady orbit is the desired motion.

**Task/Method setup**:
- *Spong 1995 (Acrobot)*: two-link underactuated robot; partial-feedback-linearisation creates unstable zero dynamics that pump energy into the passive link; a switched linear controller catches the arm at the inverted equilibrium.
- *Nakanishi et al. 2000 (brachiation)*: two-link brachiating robot; encodes pendulum-like swing as a "target dynamical system" (limit cycle); the robot tracks this attractor to achieve continuous multi-rung locomotion without per-step planning.

**Sim vs real**: Both demonstrated on physical hardware. Nakanishi's brachiation controller transferred directly from target dynamics design to real robot without separate sim-to-real gap handling.

**Core idea / mechanism**:
1. *Energy pumping*: define a scalar energy error E(q,q̇) − E* and drive it to zero via a Lyapunov-stable control law; this monotonically grows oscillation amplitude until a stable limit cycle at desired energy E* is reached.
2. *CPG / target dynamical system*: replace explicit trajectory planning with an autonomous ODE (e.g., Hopf oscillator) whose limit cycle IS the desired periodic motion; robot joints track oscillator output, inheriting orbital stability for free.
3. *Switching / catch*: once the limit cycle is established, switch to a separate stabilising controller (LQR / balance) at a trigger surface — the swing-up phase and the balance phase are decoupled.

**Why it matters for OUR problem**:
- **Wind-up / Hopf limit cycle**: The energy-pumping mechanism is the formal justification for the optional wind-up stage in our approach — a Hopf oscillator driven to E* gives a *repeatable*, open-loop-stable initial condition for the rope before release, making downstream trajectory planning model-consistent.
- **Compact action**: The CPG parameterises a whole periodic trajectory by a handful of oscillator parameters (frequency, amplitude, phase offset), directly analogous to our min-jerk via-point spline decoder — both compress action space to a low-dimensional smooth manifold.
- **Open-loop execution**: energy pumping requires no online feedback once the gain law is fixed; it is a pure feedforward policy after calibration, matching our hard constraint.
- **Sim2real**: Nakanishi's target-dynamics approach tolerates rope/link parameter mismatch because the attractor is defined by energy, not exact timing — robustness mode analogous to our meta-learned context encoder absorbing sim2real gap.
- **Forward model connection**: knowing the energy level and oscillator phase fully predicts future tip trajectory, which is the invariant our forward model must learn.

**Key result**:
- Spong: energy-based swing-up converges in a few oscillations for the Acrobot; catch-and-balance is reliable within a small basin.
- Nakanishi: continuous brachiation over multiple rungs achieved on real hardware; the target-dynamical-system controller requires only pendulum natural-frequency knowledge, not full link-parameter identification.
