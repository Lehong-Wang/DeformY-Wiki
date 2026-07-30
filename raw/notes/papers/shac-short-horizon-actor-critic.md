---
title: "Accelerated Policy Learning with Parallel Differentiable Simulation"
authors: ["Jie Xu", "Viktor Makoviychuk", "Yashraj Narang", "Fabio Ramos", "Wojciech Matusik", "Animesh Garg", "Miles Macklin"]
venue: ICLR
year: 2022
arxiv_id: "2204.07137"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# SHAC: Short-Horizon Actor-Critic

**One-line gist**: Tame chaotic/stiff gradients in parallel differentiable simulation by splitting the horizon into short sub-windows and mixing first-order actor updates with a model-free critic.

**Task/Method setup**: Robotic control benchmark (locomotion, manipulation) in a GPU-accelerated differentiable rigid-body simulator (Warp/IsaacGym). Policy is a neural network trained entirely in simulation using analytic gradients from the differentiable physics engine.

**Sim vs real**: Simulation only; no sim2real transfer studied in this paper. The differentiable sim is the training substrate.

**Core idea / mechanism**:
- Full-horizon gradient backpropagation through stiff/contact-rich physics explodes or vanishes; SHAC instead truncates at a short horizon `h ≪ H`.
- At each step, `N` short-horizon trajectories of length `h` are sampled in parallel on GPU; the actor is updated via first-order (analytic) gradients through those `h` steps only.
- A model-free critic bootstraps value estimates beyond the truncation boundary, completing the effective policy gradient without requiring gradients to flow over the full episode.
- The hybrid avoids gradient chaos while retaining the data efficiency advantage of analytic gradients vs. pure RL.

**Why it matters for OUR problem**:
- **Forward model / sim training**: Our GPU-parallel simulator is the dominant training substrate; SHAC's short-horizon trick is directly applicable to learning a forward model or policy inside that simulator without gradient explosion through the rope's stiff contact dynamics.
- **Compact action / spline decoder**: SHAC operates on parameterized actions; the same actor-critic loop can optimize spline via-point parameters rather than raw torques, keeping the action space compact.
- **Robust planning**: SHAC-style analytic gradients are faster per iteration than model-free RL for trajectory optimization in the forward model's learned latent space — useful for the PETS/diffusion planning step when the forward model is differentiable.
- **Sim2real / meta-adaptation**: Not addressed here, but the fast convergence under short-horizon gradients makes sim-side pre-training cheaper before RMA-style real calibration.

**Key result**: On six robotic benchmarks SHAC converges 10-100× faster than PPO/SAC baselines and matches or exceeds final performance, demonstrating that short-horizon analytic gradients plus a model-free critic is a robust combination for contact-rich differentiable simulation.
