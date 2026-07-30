# Query Pack (general)

_Auto-generated compressed context. Do not edit._

## Methods (10 total)
- [data] Per-Timestep Hindsight Relabeling
- [architecture] Smooth-Basis Swing Parameterization
- [architecture] Conditional Flow Matching over Motion-Primitive Parameters
- [inference] Sim-Verified Best-of-N Selection
- [evaluation] Direction-Reachability Atlas
- [inference] Diffuser: Guided Diffusion Planning
- [training] GrBAL / ReBAL: Online Meta-Learned Dynamics Adaptation
- [architecture] MMP++ / IMMP++ (Parametric-Curve Motion Manifold Primitives)
- [inference] PETS: Probabilistic Ensembles with Trajectory Sampling
- [training] RMA: Rapid Motor Adaptation (two-phase context-encoder adaptation)
## Open Gaps
_Auto-generated open questions. Do not edit._
- [paper/accurate-simulation-parameter-identification-dlos-using] How much of the static accuracy advantage transfers to **dynamic** regimes (whipping, swinging, throwing) where the linear-stiffness `native` model would similarly miss curvature-twist coupling?
- [paper/accurate-simulation-parameter-identification-dlos-using] Does the generalized-coordinate DER step compose cleanly with **MJX**'s vectorized GPU pipeline to enable batched RL rollouts at MuJoCo time scales? (Compatible in principle — same physics — but throughput at scale is unmeasured.)
- [paper/accurate-simulation-parameter-identification-dlos-using] Can a **learning-augmented** layer on top of identified $\alpha, \beta$ capture residual hysteresis / plastic effects without losing the warm-start advantage the paper emphasizes?
- [paper/accurate-simulation-parameter-identification-dlos-using] Could the same force-lever conversion be applied to higher-order rod models (Cosserat with stretch and shear) without re-introducing the Jacobian-query overhead?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] How much of APG's Whip-Rope win is due to MLS-MPM particle physics vs. due to JAX-batched gradient flow? Would the same algorithm dominate on a Cosserat-rod or linked-capsule rope?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] Can entropy-regularized differentiable RL (e.g. add policy entropy bonus to APG) clos
## Papers (23 total)
- [4] DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects
- [5] Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects
- [4] Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy
- [4] Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation
- [4] Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting
- [4] Learning to Adapt in Dynamic, Real-World Environments Through Meta-Reinforcement Learning — Meta-trains a dynamics-model prior whose update rule rapidly re-fits the model from the last M timesteps to predict the next K, giving model-based RL agents (GrBAL/ReBAL) fast online adaptation to crippled limbs, new terrains, and payloads — demonstrated on the first meta-RL real robot.
- [4] Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables
- [5] Deep Reinforcement Learning in a Handful of Trials using Probabilistic Dynamics Models — PETS combines a probabilistic ensemble of bootstrapped neural dynamics models (separating aleatoric from epistemic uncertainty) with trajectory-sampling uncertainty propagation and CEM-based MPC planning, matching model-free asymptotic returns on MuJoCo control with 8-125x fewer samples.
- [4] Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration
- [4] DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics
- [3] SoftMimicGen: A Data Generation System for Scalable Robot Learning in Deformable Object Manipulation
- [5] TossingBot: Learning to Throw Arbitrary Objects with Residual Physics
- [3] Rapid Adaptation of Particle Dynamics for Generalized Deformable Object Mobile Manipulation
- [3] Self-Curriculum Model-based Reinforcement Learning for Shape Control of Deformable Linear Objects
- [4] Learning Deformable Object Manipulation Using Task-Level Iterativ
## Recent Relationships (252 total)
  methods/per-timestep-hindsight-relabeling --derived_from--> foundations/imitation-learning
  methods/per-timestep-hindsight-relabeling --derived_from--> foundations/behavioral-cloning
  methods/smooth-basis-swing-parameterization --derived_from--> foundations/minimum-jerk-trajectory
  methods/smooth-basis-swing-parameterization --derived_from--> foundations/movement-primitives
  methods/smooth-basis-swing-parameterization --derived_from--> foundations/trajectory-optimization
  methods/conditional-flow-matching-motion-parameters --derived_from--> foundations/denoising-diffusion-probabilistic-models
  methods/conditional-flow-matching-motion-parameters --derived_from--> foundations/movement-primitives
  methods/sim-verified-best-of-n-selection --derived_from--> foundations/cross-entropy-method
  methods/sim-verified-best-of-n-selection --derived_from--> foundations/trajectory-optimization
  methods/direction-reachability-atlas --derived_from--> foundations/optimization
  ideas/directio
