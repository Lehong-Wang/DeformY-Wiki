---
title: "An Introduction to the Mechanics of the Lasso"
authors: ["Pierre-Thomas Brun", "Neil Ribe", "Basile Audoly"]
venue: "Proceedings of the Royal Society A"
year: 2014
arxiv_id: null
doi: "10.1098/rspa.2014.0512"
note_type: bibliography_only
sources: [field-research]
---

# Mechanics of the Lasso

**One-line gist**: Derives the steady-state dynamics of a rotating lasso loop (Flat Loop, Spoke, Hondo Knot) and identifies which wrist-drive kinematics produce stable orbits versus collapse.

**Task/Method setup**: Treats the lasso as an inextensible elastic rod undergoing large-amplitude planar/3D rotation driven by a prescribed wrist trajectory. Solves for equilibrium shape and linearises to assess orbital stability as a function of loop radius and drive frequency.

**Sim vs real**: Analytical (Kirchhoff rod equations) + physical rope experiments; no robot arm or GPU simulator involved.

**Core idea / mechanism**: Steady "Flat Loop" orbits exist only within a band of radii and frequencies; outside this band the loop either collapses inward or whips outward. The wrist must trace a specific elliptical figure-8 to sustain a stable orbit — essentially a Hopf-like limit cycle maintained by periodic energy injection matching dissipation. The paper characterises the bifurcation boundary analytically.

**Why it matters for OUR problem**:
- **Wind-up / limit cycle**: Directly informs the optional Hopf wind-up stage — confirms that a stable rotating initial condition exists and gives the parameter regime (radius, frequency, drive shape) where it lives. Calibrating to this regime gives a repeatable open-loop start state.
- **Compact action**: The figure-8 wrist trajectory is a low-dimensional parametric action (amplitude, frequency, phase); fits naturally into a min-jerk via-point spline decoder.
- **Forward model**: The analytical stability map can seed or constrain a learned forward model's input space, reducing the region that must be explored in sim.
- **Feasibility map**: Stability boundaries (radius × frequency) directly define which wind-up conditions are feasible before the release/swing phase.

**Key result**: A lasso loop is stable iff the wrist drives it at a frequency within ≈ [0.7, 1.3]× the natural rotating frequency for that loop radius; outside this band the orbit is unstable. Experimental validation with real rope confirms the analytical bifurcation boundaries.
