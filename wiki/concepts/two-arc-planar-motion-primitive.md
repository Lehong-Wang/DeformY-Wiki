---
title: "Two-Arc Planar Motion Primitive"
aliases: ["polar two-arc primitive", "two-arc cable casting primitive", "four-parameter polar trajectory", "wrist-rotation arc primitive", "polar arc + wrist primitive"]
tags: [DLO, dynamic-manipulation, action-parameterization, free-end-cable, motion-primitive, robot-casting]
maturity: emerging
key_papers: ["[[self-supervised-learning-dynamic-planar-manipulation]]"]
first_introduced: "2024"
date_updated: 2026-05-06
related_concepts: ["[[real2sim2real-pipeline]]"]
---

## Definition

A **low-dimensional, hand-engineered action parameterization** for dynamic planar manipulation of a free-end cable, in which a robot arm executes two consecutive sweeping arcs in polar coordinates plus an optional wrist rotation. The full action is

$$
\ba = (\theta_1, \theta_2, r_2, \psi),
$$

interpreted relative to a polar coordinate frame whose origin is offset by a fixed system parameter $r_0$ from the robot's reset pose, with a fixed maximum end-effector velocity $\vmax$. Introduced in [[self-supervised-learning-dynamic-planar-manipulation]] for free-end cable casting on a UR5.

## Intuition

Dynamic free-end cable manipulation needs a motion that (i) sweeps wide enough to cover most of the workspace semicircle, (ii) is repeatable enough that supervised learning of $f_{\rm forw}: \ba \to (x_f, y_f)$ converges with $\le 10^4$ trajectories, (iii) is interpretable so coverage and repeatability can be audited, and (iv) admits an analytic IK so deployment doesn't require trajectory optimization. Observing humans casting cables, the authors notice the motion arcs one direction, reverses, and stops — a polar two-arc trajectory. Adding a wrist rotation $\psi$ to the second arc converts a planar bang-bang sweep into a 3D scoop that broadens the reachable area without inflating the action dimension.

## Formal notation

Let $(r_0, \theta_0)$ be the reset end-effector pose in a polar frame whose origin is shifted by $r_0$ from the robot base. The end-effector trajectory passes through $(r_0,\theta_0) \to (r_1,\theta_1) \to (r_2, \theta_2)$ with:

- **Angular interpolation**: jerk-limited bang-bang at maximum velocity $\vmax$ from $\theta_0$ to $\theta_1$, then $\theta_1$ to $\theta_2$, with zero angular velocity and acceleration at $\theta_1$ enforcing a clean direction reversal.
- **Radial interpolation**: cubic spline $r_0 \to r_2$ in time, with $r_1$ implicitly defined by the spline at the time the angular trajectory hits $\theta_1$.
- **Wrist rotation**: during the second arc, the wrist joint about the $z$-axis rotates from $\theta_2$ to $\psi$ via a maximum-velocity spline ($\psi \ge \theta_2$ in canonical form; sign-flipped for the mirror-image left half-plane).

System parameters $(r_0, \vmax)$ are selected once per cable to maximize a coverage × repeatability metric and then held fixed; only $(\theta_1, \theta_2, r_2, \psi)$ vary across trials. Symmetry exploitation: training data are sampled in $\theta_1<0, \theta_2>0, \psi\ge\theta_2$ (right half-plane); left-side targets are mirrored.

## Variants

- **$A_1$ without wrist** — $\ba = (\theta_1, \theta_2, r_2)$, three parameters; simulated workspace coverage 21–66% depending on $\vmax$.
- **$A_2$ with wrist** ($\ba = (\theta_1, \theta_2, r_2, \psi)$) — four parameters; coverage 76–80% at the chosen operating point. The default in [[self-supervised-learning-dynamic-planar-manipulation]].
- **Different $r_0$ values** — $r_0=0.6$ produces tightly-curved arcs that maximize coverage; $r_0=2.0$ produces flat arcs that lose central coverage. Empirically $r_0=0.6$ wins.

## Comparison

- vs. **5-parameter PRC action** $(\theta_1, r_1, \theta_2, r_2, \alpha)$ in [[planar-robot-casting-real2sim2real-self-supervised]]: PRC explicitly samples both $r_1$ and $r_2$ as free parameters, and uses $\alpha$ as a scalar wrist twist offset rather than a terminal target wrist angle. The free-end paper drops $r_1$ as a free parameter (it's spline-implicit) in favor of the absolute wrist target $\psi$ — net dimension is 4 vs. 5. The two parameterizations are not interchangeable: the new one is tied to the right half-plane symmetry trick and a specific cubic-spline radial schedule.
- vs. **apex-point parameterization** in [[robots-lost-arc-self-supervised-learning]]: Zhang et al.'s fixed-end-cable apex-point is geometrically very different — it parameterizes the *peak* of the swing, not the angular waypoints — and assumes both cable ends are anchored, which removes the free-end's translational dynamics.
- vs. **dynamic motion primitives (DMPs)** and other generic learnable primitives: DMPs learn a dynamical system from demonstrations and are not constrained to polar coordinates or to two arcs. They are more flexible but typically need orders of magnitude more data to learn the cable-relevant region.
- vs. **direct joint-space trajectory generation**: generating a full UR5 joint trajectory (6 DOFs × $T$ timesteps) blows up the regression target; this primitive collapses to 4 scalars, and IK fills in the rest.

## When to use

- The task is **planar** dynamic manipulation of a **free-end** 1D deformable object (rope, cable, wire, thread).
- A consistent reset pose is achievable so the action's start state is canonical.
- The desired action is a single-shot dynamic motion (no closed-loop feedback during the swing).
- A small (~$10^4$) labeled dataset is realistic — the low parameter count lets supervised learning converge fast.

Avoid this primitive when you need closed-loop correction, when 3D out-of-plane dynamics matter, or when the cable's behavior depends on multiple disjoint motion modes (the two-arc form bakes in a single direction reversal).

## Known limitations

- **Hand-crafted, not learned**: assumes the human-observed two-arc motion is structurally adequate; cannot represent arbitrary dynamic motions.
- **Single direction reversal**: more complex motions (pendulum-like multi-swing, helical) cannot be expressed.
- **Symmetry exploitation limits the search**: only the right half-plane is sampled in $\dreal$, so any asymmetry (e.g. cable bias from prior plastic deformation) breaks the mirror-image deployment trick.
- **Unevaluated against learned primitives**: no comparison against learned latent action spaces or DMPs at equal data budget.
- **Tied to PyBullet's spline conventions** in [[self-supervised-learning-dynamic-planar-manipulation]]; whether the same parameterization transfers to FleX-segmented or differentiable simulators is untested.

## Open problems

- Does the four-parameter form generalize to non-rope DLOs (chains, hoses, flexible tubing) or does each new object class need its own primitive?
- A principled extension to 3D — adding an out-of-plane angle and a height variable — preserving the low-dim coverage trade-off.
- Coupling the primitive with a residual learned correction term (à la [[iterative-residual-policy-goal-conditioned-dynamic]]) to clamp the long-tail outliers seen in [[self-supervised-learning-dynamic-planar-manipulation]].
- Whether a learned primitive (small VAE or implicit motion model) trained on top of physical demonstrations beats this hand-crafted one at equal data.

## Key papers

- [[self-supervised-learning-dynamic-planar-manipulation]] — Wang et al., 2024. Introduces the four-parameter polar two-arc + wrist primitive and demonstrates it covers ~80% of the UR5's reachable semicircle while keeping the regression target small enough for supervised learning to converge on $|\dsim|=36{,}000$ trajectories.

## My understanding

The two-arc primitive is a textbook example of "the right inductive bias is half the work" in robot learning: by hand-encoding polar coordinates, jerk-limited bang-bang angular motion, cubic radial splines, and a single direction reversal at $\theta_1$, the authors collapse what would otherwise be a 6-DOF × time trajectory regression into a 4-parameter regression that an MLP can fit in minutes. The wrist rotation $\psi$ is the small but real engineering insight — it converts the planar sweep into a 3D scoop and is what gets coverage past the polar-casting baseline. The trade-off is that the primitive bakes in domain assumptions: planar workspace, single-shot motion, mirror symmetry, no closed-loop feedback. Any extension of the DeformY arc to 3D or closed-loop will need to either generalize this primitive or replace it.
