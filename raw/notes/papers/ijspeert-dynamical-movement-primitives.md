---
title: "Dynamical Movement Primitives: Learning Attractor Models for Motor Behaviors"
authors: ["Auke Jan Ijspeert", "Jun Nakanishi", "Heiko Hoffmann", "Peter Pastor", "Stefan Schaal"]
venue: "Neural Computation"
year: 2013
arxiv_id: null
doi: "10.1162/NECO_a_00393"
note_type: bibliography_only
sources: [field-research]
---

# Dynamical Movement Primitives (DMPs)

**One-line gist**: Encode any demonstrated trajectory as a stable attractor dynamical system (discrete or rhythmic) that can be scaled, temporally stretched, and goal-conditioned without re-learning.

**Task/Method setup**: A second-order spring-damper system drives a trajectory toward a goal; a nonlinear forcing function (learned via locally weighted regression from one demonstration) shapes the path. Two canonical forms — *discrete* (point attractor, reaches a fixed goal) and *rhythmic* (limit-cycle attractor, sustains periodic motion). Phase variable decouples time from motion, enabling temporal scaling.

**Sim vs real**: Method is purely analytical/learned; demonstrated on real robot arms for tasks like drumming, throwing, and bimanual manipulation. No sim required; transfers directly once forcing function is fit.

**Core idea / mechanism**: Transform any demonstrated joint or task-space trajectory into a set of ODEs whose attractor landscape guarantees convergence. The forcing function `f(x)` is a weighted sum of radial basis functions fit to the residual needed to reproduce the demo. At runtime, changing the goal parameter `g` or the temporal scaling `τ` morphs the trajectory smoothly without re-fitting.

**Why it matters for OUR problem**:
- **Compact smooth action**: DMPs are an ideal spline-like action parameterization — the BF weights form a compact, smooth, differentiable action vector exactly analogous to via-point spline decoders. Planning in weight space gives a low-dimensional search space with built-in smoothness and joint-limit-friendliness.
- **Wind-up / rhythmic primitive**: The *rhythmic* DMP variant encodes a stable limit cycle; this maps directly onto the Hopf wind-up idea for generating a repeatable open-loop initial condition before release. A single rhythmic DMP can sustain a whipping oscillation, and the discrete DMP can then snap to the target.
- **Forward-model / meta-adaptation**: DMP weights are fast to adapt via LWR from a handful of real rollouts — compatible with the few-minute real calibration budget. A context encoder (RMA-style) could output delta-weights to specialize the sim-trained prior.
- **Robust planning**: Cost-guided optimization (PETS, CEM, or diffusion) operates natively over DMP weight vectors, inheriting the smooth landscape and avoiding high-frequency action artifacts that cause model exploitation.

**Key result**: Single-stroke demonstrations reproduced with <5% end-point error across temporal scales 0.5×–2× and goal perturbations up to 50% of trajectory length; rhythmic DMPs track limit-cycle drumming stably over hundreds of cycles. Widely adopted as a community baseline for imitation and motion-planning in robotics.
