---
title: "The Coordination of Arm Movements: An Experimentally Confirmed Mathematical Model"
authors: [Tamar Flash, Neville Hogan]
venue: Journal of Neuroscience
year: 1985
arxiv_id: null
doi: 10.1523/jneurosci.05-07-01688.1985
note_type: bibliography_only
sources: [field-research]
---

# Flash & Hogan Minimum-Jerk Model

**One-line gist**: Minimising the integral of squared jerk over a fixed-duration reach yields a closed-form quintic polynomial hand trajectory that matches observed human arm movements quantitatively.

**Task/Method setup**: Planar multi-joint reaching movements recorded from human subjects. Objective: find the cost functional whose minimiser reproduces the smooth, bell-shaped velocity profiles seen empirically. The chosen cost is ∫ ||d³x/dt³||² dt (mean-squared jerk). Minimisation subject to boundary conditions (position, velocity, acceleration = 0 at start and end) produces a unique degree-5 polynomial in time for each Cartesian coordinate.

**Sim vs real**: Pure motor-neuroscience study on human subjects; no simulation. Ground truth is motion-capture of hand/arm.

**Core idea / mechanism**: The minimum-jerk quintic closed form for a point moving from x₀ to x_f in duration T is:
  x(t) = x₀ + (x_f − x₀) · [10τ³ − 15τ⁴ + 6τ⁵],  τ = t/T.
Velocity profile is symmetric and bell-shaped; jerk is bounded and smooth. Boundary conditions on velocity and acceleration at both endpoints are automatically satisfied with zero values, giving a self-contained via-point parametrisation.

**Why it matters for OUR problem**:
- **Compact smooth action**: The quintic polynomial is the canonical justification for using minimum-jerk splines as the action representation in our spline decoder. Each trajectory segment between via-points inherits this form, ensuring smoothness (bounded jerk → torque-rate bounded), low parameter count, and physical plausibility — exactly the "compact smooth action" design goal.
- **Via-point extension**: The same framework extends to via-points with unconstrained intermediate velocities/accelerations; optimising over those intermediate values is the basis for our via-point spline decoder that a planner searches over.
- **Forward model target**: Our forward model maps spline parameters → tip trajectory; minimum-jerk splines keep that input space low-dimensional and well-conditioned, reducing the variance the ensemble must cover.
- Not directly relevant to meta-adaptation, wind-up, or sim2real, but underpins the action-space design that affects all of them.

**Key result**: Minimum-jerk quintic polynomials predict straight-line hand paths with bell-shaped speed profiles for point-to-point reaches; curved paths emerge naturally for via-point movements. Model fits experimental data (velocity profiles, path curvature) with high fidelity across subjects and movement durations.
