# Query Pack (general)

_Auto-generated compressed context. Do not edit._

## Methods (15 total)
- [data] Per-Timestep Hindsight Relabeling
- [architecture] Smooth-Basis Swing Parameterization
- [architecture] Conditional Flow Matching over Motion-Primitive Parameters
- [inference] Sim-Verified Best-of-N Selection
- [evaluation] Direction-Reachability Atlas
- [architecture] DA-MMP (Dynamics-Aware Motion Manifold Primitives)
- [inference] Diffuser: Guided Diffusion Planning
- [architecture] Diffusion Policy (Visuomotor Action Diffusion)
- [protocol] DLO Agent (VLM Grasp Proposal + Agentic Task Decomposition)
- [architecture] DMMP / DMMFP + TMO (Differentiable Motion Manifold Primitives)
- [training] GrBAL / ReBAL: Online Meta-Learned Dynamics Adaptation
- [architecture] MMFP (Motion Manifold Flow Primitives)
- [architecture] MMP++ / IMMP++ (Parametric-Curve Motion Manifold Primitives)
- [inference] PETS: Probabilistic Ensembles with Trajectory Sampling
- [training] RMA: Rapid Motor Adaptation (two-phase context-encoder adaptation)
## Open Gaps
_Auto-generated open questions. Do not edit._
- [paper/accurate-simulation-parameter-identification-dlos-using] How much of the static accuracy advantage transfers to **dynamic** regimes (whipping, swinging, throwing) where the linear-stiffness `native` model would similarly miss curvature-twist coupling?
- [paper/accurate-simulation-parameter-identification-dlos-using] Does the generalized-coordinate DER step compose cleanly with **MJX**'s vectorized GPU pipeline to enable batched RL rollouts at MuJoCo time scales? (Compatible in principle — same physics — but throughput at scale is unmeasured.)
- [paper/accurate-simulation-parameter-identification-dlos-using] Can a **learning-augmented** layer on top of identified $\alpha, \beta$ capture residual hysteresis / plastic effects without losing the warm-start advantage the paper emphasizes?
- [paper/accurate-simulation-parameter-identification-dlos-using] Could the same force-lever conversion be applied to higher-order rod models (Cosserat with stretch and shear) without re-introducing the Jacobian-query overhead?
- [paper/da-mmp-learning-coordinated-accurate-throwing] Would per-timestep or per-waypoint outcome labels beat one landing label per trial? Each executed throw currently yields exactly one training pair; the ring's whole flight path is measured but discarded.
- [paper/da-mmp-learning-coordinated-accurate-throwing] Does best-of-N sampling with a simulator verifier lift the 60% materially, and at what per-target compute 
## Papers (28 total)
- [4] DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects
- [5] Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects
- [4] Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy
- [4] Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation
- [4] Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting
- [4] DLO-Lab: Benchmarking Deformable Linear Object Manipulations with Differentiable Physics — A differentiable Discrete-Elastic-Rods simulator for deformable linear objects built inside Genesis/Taichi — the first to combine differentiability with two-way multi-material coupling, bending plasticity and loop topology — plus a 10-task manipulation benchmark on which gradient-free CMA-ES trajectory optimization (86.6% average success) decisively beats analytic-gradient and model-free RL methods.
- [4] Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration
- [4] Learning to Adapt in Dynamic, Real-World Environments Through Meta-Reinforcement Learning — Meta-trains a dynamics-model prior whose update rule rapidly re-fits the model from the last M timesteps to predict the next K, giving model-based RL agents (GrBAL/ReBAL) fast online adaptation to crippled limbs, new terrains, and payloads — demonstrated on the first meta-RL real robot.
- [4] Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables
- [4] DA-MMP: Learning Coordinated and Accurate Throwing with Dynamics-Aware Motion Manifold Primitives — Extends motion manifold primitives to variable-length trajectories via a via-point RBF parameterization with duration as an explicit parameter, autoencodes 90k sampling-planner-generated ring-toss trajectories into a 64-D latent manifold, and trains a conditional flow-matching model in that latent space on only 60 execut
## Recent Relationships (304 total)
  papers/dlo-lab-benchmarking-deformable-linear-object --extends_concept--> concepts/differentiable-deformable-benchmark
  papers/dlo-lab-benchmarking-deformable-linear-object --extends_concept--> concepts/differentiable-discrete-elastic-rods
  papers/dlo-lab-benchmarking-deformable-linear-object --extends_concept--> concepts/cosserat-isaac-cosimulation
  papers/dlo-lab-benchmarking-deformable-linear-object --introduces_concept--> concepts/gradient-inaccessibility-contact-mediated-manipulation
  papers/dlo-lab-benchmarking-deformable-linear-object --derived_from--> foundations/discrete-elastic-rods
  papers/dlo-lab-benchmarking-deformable-linear-object --derived_from--> foundations/cosserat-rod-theory
  papers/dlo-lab-benchmarking-deformable-linear-object --derived_from--> foundations/position-based-dynamics
  papers/dlo-lab-benchmarking-deformable-linear-object --derived_from--> foundations/deformable-linear-object
  papers/dlo-lab-benchmarking-deformable-linear-object --derived_from-
