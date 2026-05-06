---
title: "DER-MuJoCo Generalized-Coordinate Coupling"
aliases: ["adapted DER in MuJoCo", "generalized-coordinate DER", "force-lever DER adaptation", "DER-MuJoCo coupling", "DER in joint coordinates"]
tags: [DLO, simulation, discrete-elastic-rods, mujoco, generalized-coordinates, force-lever]
maturity: emerging
key_papers: ["[[accurate-simulation-parameter-identification-dlos-using]]"]
first_introduced: "2025"
date_updated: 2026-05-06
related_concepts: []
---

## Definition

DER-MuJoCo generalized-coordinate coupling is an architectural pattern that embeds Bergou et al.'s **Discrete Elastic Rods** bending and twisting energies inside MuJoCo by converting the DER Cartesian centerline forces into ball-joint torques in MuJoCo's generalized coordinates via a moment-based force-lever analysis. The DLO is a chain of MuJoCo capsules connected by ball joints (rotation only); per-step DER node forces $\vec{F}_j$ are mapped to joint torques $\vec{T}_i = \sum_j \vec{F}_j \times \vec{D}_{i,j}$ (with the sign convention chosen so the parent→child kinematic-tree orientation MuJoCo requires is respected) and applied via `qfrc_passive`. Inextensibility is enforced by MuJoCo's existing constraint solver. The material-frame twist is treated **quasi-statically**, mirroring the DER theory's closed-form Gauss-Seidel-style update.

## Intuition

Two representations of an elastic rod live at different speeds in MuJoCo. (i) The DER theory naturally expresses stiffness as a *Cartesian* force at every centerline node, the negative of the energy gradient. Applying these directly inside MuJoCo would require querying $n+1$ separate Jacobians per step — expensive and antithetical to MuJoCo's generalized-coordinate solver. (ii) MuJoCo's native cable model expresses stiffness as a *linear* torque-vs-deformation law at every ball joint — cheap, but unfaithful to Kirchhoff rod mechanics at large deformations and prone to dynamic twist-wave artifacts because it integrates the twist degree of freedom dynamically rather than quasi-statically. The coupling pattern keeps DER's (correct) energy expressions but transports them into joint coordinates with one Jacobian-free step: each Cartesian force is converted into the joint torques it would induce by simple lever-arm summation across the kinematic chain. The net stiffness force on a rod is zero, the chain consists of ball joints, and torque application across a parent-child boundary is its own negative — these three observations make the conversion cheap and exact for the static / quasi-static regime.

## Formal notation

Given $n+1$ DER nodes with Cartesian forces $\vec{F}_j$ from the energy gradient,

$$\vec{T}_i = \sum_{j=0}^{n+1} \vec{F}_j \times \vec{D}_{i,j}, \quad \vec{D}_{x,y} = \begin{cases} \vec{X}_{x,y}, & x < y \\ \vec{X}_{y,x}, & \text{otherwise}\end{cases}$$

where $\vec{X}_{i,j}$ is the position vector from joint $i$ to joint $j$. The torque $\vec{T}_i$ is applied at joint $i$ via MuJoCo's `qfrc_passive`. Standard MuJoCo joint damping handles dissipation; the centerline twist $\theta^j$ is updated each step as the energy-minimizing twist for the current centerline (no axial-rotation damping required).

## Variants

- **Direct DER in MuJoCo Cartesian frame** (`direct`): apply DER Cartesian forces directly to body COMs; needs $n+1$ Jacobians per step. Same accuracy as the adapted variant but $\sim 5\times$ slower per step at large $n$.
- **Adapted DER (force-lever) in generalized coordinates** (`adapted`): the variant defined here.
- **Differentiable / GPU MJX port** (open): DER step wrapped in MJX-style vectorization for batched policy training. Compatible in principle, untested in the original paper.

## Comparison

- vs. **MuJoCo native cable**: native applies a *linear* torque response per ball joint (no curvature-twist coupling, no quasi-static twist treatment). Static error in Michell's instability test is $\sim 16\times$ larger; native exhibits unnatural twist-wave oscillations that can dominate the error budget on poses with large axial twist. Computational cost is similar.
- vs. **stand-alone DER (PyElastica)**: stand-alone solvers achieve the same physics but live outside MuJoCo, so contact with rigid bodies and a robot requires bespoke coupling code. The adapted pattern keeps everything inside MuJoCo's existing constraint and contact pipeline.
- vs. **[[cosserat-isaac-cosimulation]]** (DeformX): co-simulation pairs a dedicated rod engine with Isaac Sim via multi-rate impulse exchange — strictly more expressive (Cosserat with stretch/shear, free-form mesh contact), but heavier to deploy. The DER-MuJoCo coupling is the lighter alternative when the DLO can be assumed inextensible and unsheared.
- vs. **mass-spring** / **PBD-style** rod models in robotics simulators: cheaper but lack the explicit material-parameter interpretation that DER provides via $\alpha = EI$, $\beta = GJ_T$.

## When to use

- The DLO can reasonably be assumed inextensible and unsheared (rope, wire harness, suture).
- The downstream pipeline already lives in MuJoCo (planner / RL / MPC), and pulling in a second simulator would be expensive.
- Static or quasi-dynamic accuracy on bending and twisting matters more than fast-tip dynamics; or the user is willing to extrapolate the static accuracy to dynamic regimes where DER theory still applies.
- Parameter identification of real DLOs is needed (the same chain runs the simulated side of the depth-camera identification pipeline).

Skip this pattern if you need stretch/shear, full Cosserat dynamics, contact-rich free-form meshes, or end-to-end differentiability for trajectory optimization (use [[cosserat-isaac-cosimulation]] or [[deform-differentiable-discrete-elastic-rods-real]] instead).

## Known limitations

- CPU-only single-environment in the original implementation; batched RL-scale rollouts not demonstrated.
- Validation in the source paper covers static and quasi-dynamic regimes only; transfer to fast-tip-velocity dynamics is plausible (DER physics is correct for those regimes too) but not measured.
- Plastic deformation and hysteresis are out of scope; the coupling assumes elastic recovery to DER-energy minima.
- Sequential parent→child ball-joint chain encodes the kinematic tree; non-tree topologies (closed loops, branching DLOs) require additional constraints that the original force-lever conversion does not handle by itself.

## Open problems

- GPU / MJX-batched port of the adapted DER step at scale; required for on-policy RL on dynamic DLO tasks.
- Differentiable variant for use in trajectory optimization and analytical-gradient policy gradient methods.
- Extension to Cosserat (stretch/shear) without re-introducing the per-node Jacobian queries the force-lever step was designed to avoid.
- Online identification of $\alpha, \beta$ co-evolving with policy training, beyond the offline pipeline of the source paper.

## Key papers

- [[accurate-simulation-parameter-identification-dlos-using]] — defines the coupling, validates against helical-buckling and Michell-instability tests, and pairs it with a depth-camera + 3D-printed twist-apparatus parameter identification pipeline.

## My understanding

This is the lightest plausible substrate for getting DER-grade DLO statics into a MuJoCo-native robot-learning pipeline. For a project like DeformY whose stack is centered on MuJoCo / MJX, this pattern is more attractive than co-simulation because it requires no second engine and no multi-rate scheduling. The two open levers — MJX batching and differentiability — would directly unlock dynamic-regime sim2real and gradient-based control on top of the same physics, and both look feasible since the per-step computation is small and Jacobian-free.
