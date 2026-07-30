---
title: "Pullback Tube Acceleration"
aliases: ["pullback tube", "closed-loop tube acceleration", "pullback BRT control", "feedback tube acceleration"]
tags: [robust-control, throwing, convex-optimization, backward-reachable-tube, real-time-MPC, release-uncertainty]
maturity: emerging
key_papers: ["[[learning-accurate-whole-body-throwing-high]]"]
first_introduced: "2025"
date_updated: 2026-05-06
related_concepts: []
parent_topic: "[[dynamic-throwing-and-hitting]]"
---

## Definition

**Pullback tube acceleration** is a closed-loop release-time controller for stochastic prehensile throwing. At every fast-loop step during the 100 ms gripper-detach window, it solves a small convex program — parameterized on the *measured* end-effector (EE) state — that returns a constant-acceleration command which (i) drives the EE state toward the **backward reachable tube (BRT)** of valid release configurations for the desired landing target, and (ii) is dynamically feasible (velocity and acceleration limits respected). The BRT acts as an *attracting invariant set*: regardless of where the EE actually is when release timing is finally resolved, the controller has been pulling the state toward a configuration whose subsequent ballistic flight lands inside the user-specified target set.

The contrast with the antecedent (open-loop) tube acceleration of Liu and Billard (2024): the open-loop tube is *planned* against the *nominal* release state computed before the throw begins, then the manipulator tracks the planned tube; pullback tube acceleration is *replanned* every 0.4 ms against the *current* release state, which is essential when the upstream tracking controller (RL policy, MPC) leaves residual error.

## Intuition

For projectile throwing, every release state $(\bm{p}_d, \bm{v}_d) \in \mathbb{R}^6$ that lands inside the target set $\mathcal{X}$ forms a connected subset of state space, the BRT $\mathcal{G}$. Two facts make pullback tube acceleration work:

1. **The BRT is a connected set with no holes** (Khalil's Th. 3.5 on smooth flowmaps). So a feedback policy that *gradient-descends a distance to* the BRT can pull the EE state into it without getting stuck on disconnected components.
2. **The flowmap from $(\bm{p}_d, \bm{v}_d)$ to the landing position is locally affine** (linearizing the projectile flight around the predicted release state gives a Jacobian, used as a constraint in the convex program).

Together, these turn closed-loop release-time robustness into a *parametric convex QP* whose decision variable is the constant tube acceleration $\bm{a}_{tube}$, solvable in 0.4 ms — well below the 50–100 ms detach window — so the controller can run at >1 kHz and continuously refine the acceleration command as new state estimates arrive.

## Formal notation

Let $(\bm{p}_{EE}, \bm{v}_{EE})$ be the live measured EE state at the start of the release window, $T$ the time remaining until window terminus, and $z_{land}$ the target ground height. The pullback program is:

$$
\begin{aligned}
\min_{\bm{a}_{tube}} \quad & \left| r_{land} - r_{target} \right|^2 \\
\text{s.t.} \quad & \bm{p}_T = \bm{p}_{EE} + T \bm{v}_{EE} \\
& \bm{v}_T = \bm{v}_{EE} + T \bm{a}_{tube} \\
& \dot{r}_T = \|\bm{v}_{T,xy}\|_2,\ \dot{z}_T = \bm{v}_{T,z} \\
& r_{land}^0 = \Phi(\bm{p}_T, \bm{v}_{EE}, z_{land}) \\
& r_{land} = r_{land}^0 + \nabla_{(\dot{r},\dot{z})} r_{land}^0 \cdot ([\dot{r}_T - \dot{r}, \dot{z}_T - \dot{z}]^\top) \\
& \bm{v}_{\min} \le \bm{v}_T \le \bm{v}_{\max},\ \bm{a}_{\min} \le \bm{a}_{tube} \le \bm{a}_{\max}
\end{aligned}
$$

where $\Phi$ is the projectile-flight flowmap. All equality constraints are linear in $\bm{a}_{tube}$ and inequality constraints are box-polytopic, so the problem is a convex QP. Implemented in Disciplined Parametrized Programming (DPP) form via CVXPYgen, with parameters $(\bm{p}_{EE}, \bm{v}_{EE}, T, r_{target}, z_{land})$ updated every fast-loop step.

## Variants

- **Projectile-flight pullback** (this paper): closed-form $\Phi$ for gravity-only flight; the BRT and its flowmap admit an analytic expression.
- **Drag-aware pullback**: replace $\Phi$ with a numerically integrated flight model. Adds offline integration cost; convex form preserved if $\Phi$'s linearization is precomputed.
- **Neural-Event-ODE BRT** (proposed by Liu and Billard 2024 for the open-loop case): neural implicit representation of the BRT for arbitrary nonlinear flight dynamics — extendable to a pullback variant.
- **Multi-target pullback**: minimize over a set of acceptable targets; trivially convex with multiple `r_land` terms.

## Comparison

- vs. **open-loop tube acceleration** (Liu and Billard 2024): pullback is closed-loop on EE state and provably renders the BRT an attracting invariant set; open-loop assumes a perfect tracker and fails when residual EE error pulls the actual release state outside the planned BRT.
- vs. **online MPC over a long horizon**: pullback solves a single short-horizon (≤100 ms) convex program; full MPC over the whole throw is unnecessary because the *nominal* policy + residual already handle the pre-release motion.
- vs. **domain-randomization on release timing during training**: DR learns a single mean-best behavior; pullback is reactive and target-conditioned, so it generalizes to new targets and EE states without retraining.
- vs. **manual release-timing tuning** (TossingBot-style): hand-tuning a release time is brittle to gripper variability; pullback continuously absorbs release uncertainty by *moving* through valid states for the entire window.

## When to use

- Release timing is uncertain (gripper variability, deformable object detachment).
- The flight dynamics admit a tractable flowmap (analytic for projectiles; numerical for drag; learned for complex flight).
- A fast-rate state estimate of the EE is available (≥100 Hz; better at 400 Hz+).
- An upstream tracker (RL nominal + residual, or MPC) has bounded but non-zero residual EE error.

Skip when release timing is precisely controllable (vacuum grippers, magnetic detach), when flight dynamics are essentially intractable (highly turbulent flight regimes), or when target accuracy is loose enough that ignoring release uncertainty is fine.

## Known limitations

- Requires a tractable BRT and a flight flowmap; for spin-coupled or aerodynamically complex flight, computing or learning the BRT is itself a research problem.
- The BRT linearization is local — large EE errors at the start of the release window may exit the validity region of the affine $\Phi$ approximation.
- Assumes a single target. Multi-target generalization is convex but not yet validated.
- Object inertia is treated as small relative to EE inertia; heavy objects break the assumption.

## Open problems

- Learned flight flowmaps for non-projectile dynamics (deformable mid-flight, soft-body, articulated).
- Differentiable pullback QP composed with a residual policy, training the residual end-to-end against tube cost.
- Pullback tube acceleration for non-prehensile (e.g. cable-tip striking, whipping a payload) where the "release" is not a discrete grip-open event but a continuous transfer of momentum.
- Provable bounds on the EE error margin under which pullback's closed-loop guarantees remain valid.

## Key papers

- [[learning-accurate-whole-body-throwing-high]] — introduces pullback tube acceleration, the closed-loop convex variant of Liu and Billard's open-loop tube; demonstrates monotonic landing-error reduction with control rate (96.8 cm → 31.1 cm at 400 Hz on a 1500-condition simulation grid) and 0.276 m mean landing error at 6 m on hardware.

## My understanding

Pullback tube acceleration is the load-bearing piece of architecture that lets a learned policy stack actually *land* objects accurately under release uncertainty. Reading it from outside the throwing literature: it is the *terminal phase* of an MPC stack, restricted to a 100 ms window, restricted to a single decision variable (constant acceleration), and made fast enough (0.4 ms) to run as a feedback loop rather than a one-shot plan. For the dynamic deformable manipulation regime (DeformY's whipping/striking variants), the equivalent question is whether a tractable BRT can be defined for *cable-tip arrival* dynamics; if yes, the same architectural template — RL nominal + HF residual + tube QP — should transfer.
