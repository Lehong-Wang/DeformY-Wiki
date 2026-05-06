# Wiki Index

papers:
  - slug: deformx-versatile-co-simulation-framework-deformable
    title: "DeformX: A Versatile Co-Simulation Framework for Deformable Linear Objects"
    tags: [DLO, deformable-linear-object, cosserat-rod, simulation, sim-to-real, robot-learning, dataset, isaac-sim]
    importance: 3
    domain: Robotics
  - slug: iterative-residual-policy-goal-conditioned-dynamic
    title: "Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects"
    tags: [DLO, deformable-linear-object, dynamic-manipulation, sim-to-real, residual-learning, iterative-control, rope-whipping, cloth-placement, robot-learning, mujoco]
  - slug: tossingbot-learning-throw-arbitrary-objects-residual
    title: "TossingBot: Learning to Throw Arbitrary Objects with Residual Physics"
    tags: [robotics, manipulation, throwing, residual-physics, grasping, deep-learning, sim-to-real, self-supervised, pick-and-place]
    importance: 5
  - slug: deform-differentiable-discrete-elastic-rods-real
    title: "DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects"
    tags: [DLO, deformable-linear-object, discrete-elastic-rods, differentiable-simulation, residual-learning, position-based-dynamics, sim-to-real, robot-learning, system-identification]
  - slug: accurate-simulation-parameter-identification-dlos-using
    title: "Accurate Simulation and Parameter Identification of Deformable Linear Objects using Discrete Elastic Rods in Generalized Coordinates"
    tags: [DLO, deformable-linear-object, discrete-elastic-rods, mujoco, simulation, parameter-identification, bending-stiffness, twisting-stiffness]
  - slug: planar-robot-casting-real2sim2real-self-supervised
    title: "Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting"
    tags: [DLO, deformable-linear-object, robot-casting, sim-to-real, real2sim2real, self-supervised, isaac-gym, pybullet, differential-evolution, autolab]
  - slug: dynamic-manipulation-deformable-objects-3d-simulation
    title: "Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy"
    tags: [DLO, deformable-linear-object, diffusion-policy, test-time-adaptation, reduced-order-model, GVS, cosserat-rod, benchmark, 3D-rope-manipulation, simulation, sim-only]
  - slug: learning-accurate-whole-body-throwing-high
    title: "Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration"
    tags: [throwing, whole-body-control, legged-manipulator, residual-policy, MPC, reinforcement-learning, sim-to-real, ANYmal, dynamic-manipulation, robust-control]
  - slug: implicit-physics-aware-policy-dynamic-manipulation
    title: "Implicit Physics-aware Policy for Dynamic Manipulation of Rigid Objects via Soft Body Tools"
    tags: [robotics, deformable-linear-objects, dynamic-manipulation, soft-tool-use, system-identification, one-shot, sim-to-real, heterogeneous-system]
  - slug: learning-deformable-object-manipulation-using-task
    title: "Learning Deformable Object Manipulation Using Task-Level Iterative Learning Control"
    tags: [DLO, rope, dynamic-manipulation, iterative-learning-control, ILC, real-world-learning, flying-knot, knot-tying, xArm-7, model-based, single-demo, transfer, robot-learning]
  - slug: daxbench-benchmarking-deformable-object-manipulation-differentiable
    title: "DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics"
    tags: [DLO, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning, APG, SHAC, PPO, sim-to-real]
  - slug: wiggle-go-system-identification-zero-shot
    title: "Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation"
    tags: [DLO, deformable-linear-object, rope-manipulation, system-identification, zero-shot, sim-to-real, dynamic-manipulation, CMA-ES, trajectory-optimization, drake, xarm7, cmu]
  - slug: robots-lost-arc-self-supervised-learning
    title: "Robots of the Lost Arc: Self-Supervised Learning to Dynamically Manipulate Fixed-Endpoint Cables"
    tags: [DLO, dynamic-manipulation, self-supervised, behavior-cloning, apex-point, minimum-jerk, UR5, fixed-endpoint-cable]
    importance: 4
    domain: Robotics

concepts:
  - slug: cosserat-isaac-cosimulation
    title: "Cosserat-Isaac Co-Simulation"
    tags: [DLO, simulation, cosserat-rod, isaac-sim, sim-to-real, robot-learning]
    maturity: emerging
  - slug: iterative-residual-policy
    title: "Iterative Residual Policy"
    tags: [DLO, dynamic-manipulation, residual-learning, online-adaptation, sim-to-real, robot-learning]
    maturity: emerging
  - slug: delta-dynamics-network
    title: "Delta Dynamics Network"
    tags: [DLO, dynamic-manipulation, residual-learning, learned-dynamics, sim-to-real, robot-learning]
    maturity: emerging
  - slug: residual-physics
    title: "Residual Physics"
    tags: [robotics, hybrid-controller, residual-learning, sim-to-real, action-space, manipulation, throwing]
    maturity: active
  - slug: throwing-motion-primitive
    title: "Throwing Motion Primitive"
    tags: [robotics, motion-primitive, throwing, dynamic-manipulation, manipulation, end-effector-trajectory]
    maturity: active
  - slug: physics-informed-action-prior
    title: "Physics-Informed Action Prior"
    tags: [robotics, hybrid-controller, model-based, residual-learning, sim-to-real, action-prior]
    maturity: active
  - slug: differentiable-discrete-elastic-rods
    title: "Differentiable Discrete Elastic Rods (DDER)"
    tags: [DLO, simulation, discrete-elastic-rods, differentiable-simulation, system-identification]
    maturity: emerging
  - slug: neural-residual-on-physics-model
    title: "Neural Residual on a Physics Model"
    tags: [residual-learning, physics-informed-learning, simulation, hybrid-model, differentiable-simulation]
    maturity: active
  - slug: momentum-preserving-pbd-inextensibility
    title: "Momentum-Preserving PBD Inextensibility"
    tags: [DLO, simulation, position-based-dynamics, inextensibility, momentum-conservation, discrete-elastic-rods]
  - slug: der-mujoco-generalized-coordinate-coupling
    title: "DER-MuJoCo Generalized-Coordinate Coupling"
    tags: [DLO, simulation, discrete-elastic-rods, mujoco, generalized-coordinates, force-lever]
  - slug: real2sim2real-pipeline
    title: "Real2Sim2Real Pipeline"
    tags: [DLO, sim-to-real, system-identification, self-supervised, robot-learning]
    maturity: emerging
  - slug: differential-evolution-sim-tuning
    title: "Differential Evolution Simulator Tuning"
    tags: [sim-to-real, system-identification, optimization, simulator-tuning, DLO]
  - slug: dynamics-informed-diffusion-policy
    title: "Dynamics-Informed Diffusion Policy (DIDP)"
    tags: [DLO, diffusion-policy, imitation-learning, trajectory-optimization, reduced-order-model, robot-learning]
    maturity: emerging
  - slug: physics-informed-test-time-adaptation
    title: "Physics-Informed Test-Time Adaptation (PITA) for Diffusion Sampling"
    tags: [diffusion-policy, test-time-adaptation, score-based-sampling, physics-prior, robot-learning, DLO]
    maturity: emerging
  - slug: reduced-order-gvs-model
    title: "Reduced-Order GVS (Geometric Variable Strain) Model"
    tags: [DLO, cosserat-rod, simulation, reduced-order-model, differentiable-simulation, soft-robotics]
  - slug: high-frequency-residual-policy
    title: "High-frequency Residual Policy"
    tags: [residual-policy, reinforcement-learning, whole-body-control, fast-feedback, sim-to-real]
    maturity: emerging
  - slug: pullback-tube-acceleration
    title: "Pullback Tube Acceleration"
    tags: [robust-control, throwing, convex-optimization, backward-reachable-tube, real-time-MPC, release-uncertainty]
    maturity: emerging
  - slug: whole-body-prehensile-throwing
    title: "Whole-body Prehensile Throwing"
    tags: [throwing, whole-body-control, legged-manipulator, loco-manipulation, dynamic-manipulation]
  - slug: implicit-system-identification
    title: "Implicit System Identification"
    tags: [system-identification, implicit-encoding, sim-to-real, robot-learning, physics-aware, deformable-objects]
    maturity: emerging
  - slug: heterogeneous-soft-rigid-system
    title: "Heterogeneous Soft-Rigid System"
    tags: [deformable-linear-objects, rigid-bodies, dynamic-manipulation, heterogeneous-system, robot-learning, multi-body-coupling]
    maturity: emerging
  - slug: task-level-iterative-learning-control
    title: "Task-Level Iterative Learning Control"
    tags: [iterative-learning-control, ILC, control, deformable-object-manipulation, robot-learning, real-world-learning]
    maturity: emerging
  - slug: critical-point-objective
    title: "Critical-Point Objective"
    tags: [iterative-learning-control, ILC, control, deformable-object-manipulation, trajectory-cost, behavior-cloning, robot-learning]
    maturity: emerging
  - slug: optimization-based-inverse-model
    title: "Optimization-Based Inverse Model (Norm-Optimal ILC)"
    tags: [iterative-learning-control, ILC, control, optimization, quadratic-program, robot-learning, inverse-model]
    maturity: active
  - slug: differentiable-deformable-benchmark
    title: "Differentiable Deformable-Object Benchmark"
    tags: [DLO, DOM, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning]
  - slug: task-agnostic-system-identification
    title: "Task-Agnostic System Identification"
    tags: [DLO, system-identification, rope-manipulation, sim-to-real, zero-shot, dynamic-manipulation, neural-network-sysid]
  - slug: apex-point-trajectory-parameterization
    title: "Apex-Point Trajectory Parameterization"
    tags: [DLO, dynamic-manipulation, action-representation, trajectory-optimization, low-dimensional-action]
    maturity: emerging

topics:

people:
  - slug: cheng-chi
    name: "Cheng Chi"
    affiliation: "Columbia University (at time of publication); Stanford University (subsequent)"
    tags: [robot-learning, manipulation, deformable-objects, diffusion-policy, dynamic-manipulation]
  - slug: shuran-song
    name: "Shuran Song"
    affiliation: "Stanford University (current); Columbia University (at time of publication)"
    tags: [robot-learning, manipulation, deformable-objects, perception-for-action, dynamic-manipulation]
  - slug: andy-zeng
    name: "Andy Zeng"
    affiliation: "Google DeepMind"
    tags: [robotics, deep-learning, manipulation, vision, robot-learning, foundation-models]
  - slug: yizhou-chen
    title: "Yizhou Chen"
    affiliation: "Department of Robotics, University of Michigan, Ann Arbor (ROAHM Lab)"
  - slug: qi-jing-chen
    name: "Qi Jing Chen"
    affiliation: "Nanyang Technological University"
  - slug: vincent-lim
    title: "Vincent Lim"
    affiliation: "AUTOLAB, UC Berkeley (at time of paper)"
  - slug: guanzhou-lan
    title: "Guanzhou Lan"
    affiliation: "Northwestern Polytechnical University"
  - slug: yuntao-ma
    name: "Yuntao Ma"
    affiliation: "Robotic Systems Lab, ETH Zürich"
    tags: [legged-robotics, whole-body-control, residual-policy, sim-to-real, loco-manipulation]
  - slug: zixing-wang
    name: "Zixing Wang"
    affiliation: "Purdue University, Department of Computer Science"
    tags: [robotics, deformable-linear-objects, dynamic-manipulation, soft-tool-use, manipulation-learning]
  - slug: krishna-suresh
    name: "Krishna Suresh"
    affiliation: "Carnegie Mellon University"
    tags: [robot-learning, iterative-learning-control, ILC, deformable-object-manipulation, rope-manipulation]
  - slug: siwei-chen
    name: "Siwei Chen"
    affiliation: "National University of Singapore (AdaComp Lab)"
    tags: [deformable-object-manipulation, differentiable-physics, JAX, MPM, RL, robot-learning, benchmark]
  - slug: arthur-jakobsson
    title: "Arthur Jakobsson"
  - slug: krishna-suresh
    title: "Krishna Suresh"
  - slug: harry-zhang
    name: "Harry Zhang"
    affiliation: "AUTOLab, UC Berkeley"
    tags: [robotics, deformable-object-manipulation, dynamic-manipulation, self-supervised-learning]

ideas:

experiments:

claims:
  - slug: cosserat-physics-narrows-dlo-swinging-sim2real
    title: "Cosserat physics narrows the DLO swinging sim-to-real gap"
    tags: [DLO, sim-to-real, cosserat-rod, robot-learning, PPO]
    status: weakly_supported
    confidence: 0.55
    domain: Robotics
  - slug: irp-zero-shot-sim2real-rope-whipping
    title: "Iterative Residual Policy enables zero-shot sim-to-real dynamic rope-tip targeting on UR5 across diverse rope properties and robot embodiments"
    tags: [DLO, sim-to-real, dynamic-manipulation, rope-whipping, residual-learning, zero-shot, robot-embodiment]
    status: supported
    confidence: 0.85
    domain: Robotics
  - slug: delta-dynamics-generalizes-better-than-sysid
    title: "Learning delta dynamics from observed trajectories generalizes better than explicit system identification for dynamic deformable manipulation"
    tags: [DLO, sim-to-real, residual-learning, system-identification, delta-dynamics, robot-learning]
    status: weakly_supported
    confidence: 0.7
    domain: Robotics
  - slug: residual-physics-improves-throw-accuracy
    title: "Learning a residual on top of an analytic ballistic prior yields high-accuracy goal-conditioned throwing of arbitrary objects"
    tags: [robotics, throwing, residual-physics, manipulation, sim-to-real, hybrid-controller]
    status: supported
    confidence: 0.85
  - slug: differentiable-der-plus-residual-realtime-dlo
    title: "Differentiable DER plus a neural residual achieves real-time, accurate dynamic DLO modeling, perception, and shape-matching control on real robots"
    tags: [DLO, simulation, differentiable-simulation, residual-learning, sim-to-real, real-time, perception, manipulation]
    status: supported
    confidence: 0.75
    domain: Robotics
  - slug: der-mujoco-improves-static-dlo-accuracy-over-native-cable
    title: "Integrating DER bending+twisting energies into MuJoCo's generalized-coordinate solver yields more accurate static / quasi-dynamic DLO simulation than MuJoCo's native cable plugin"
    tags: [DLO, simulation, discrete-elastic-rods, mujoco, accuracy, sim-to-real]
    status: weakly_supported
    confidence: 0.7
  - slug: real2sim2real-prc-tip-error-8-14-percent
    title: "Real2Sim2Real reaches 8–14% median tip-error on planar robot casting"
    tags: [DLO, sim-to-real, real2sim2real, robot-casting, free-end-cable, supervised-learning]
    status: supported
    confidence: 0.8
  - slug: didp-3d-rope-tip-targeting-success-rates
    title: "DIDP reaches 84.3% within 5 cm and 20.8% within 1 cm tip-to-goal on a 3D reduced-order Cosserat rope-whipping benchmark"
    tags: [DLO, diffusion-policy, benchmark, success-rate, sim-only, 3D-rope-manipulation, GVS]
    status: supported
    confidence: 0.7
    domain: Robotics
  - slug: physics-informed-tta-improves-diffusion-policy-on-dynamic-dlo
    title: "Physics-informed test-time adaptation during diffusion sampling materially improves over plain diffusion policy on dynamic DLO tasks"
    tags: [diffusion-policy, test-time-adaptation, physics-prior, DLO, sim-only, robot-learning]
    status: weakly_supported
    confidence: 0.6
    domain: Robotics
  - slug: hf-residual-tube-stack-enables-accurate-whole-body-throwing
    title: "100 Hz nominal MPC + 400 Hz RL residual + pullback-tube optimizer enables 0.276 m mean landing error at 6 m on ANYmal-D + DynaArm"
    tags: [throwing, residual-policy, whole-body, pullback-tube, ANYmal, sim-to-real]
    status: supported
    confidence: 0.75
    domain: Robotics
  - slug: legged-base-contributes-major-angular-impulse-in-throwing
    title: "Whole-body coordination contributes ~53% additional angular impulse over arm-only throwing on a legged mobile manipulator"
    tags: [throwing, whole-body, base-motion, legged-manipulator, angular-impulse, loco-manipulation]
    status: supported
    confidence: 0.7
  - slug: implicit-sysid-enables-one-shot-rope
    title: "Implicit sysID encoder + goal-conditioned action predictor enables one-shot real-deployment for rope-as-tool 3D-target transport of rigid payloads"
    tags: [implicit-system-identification, rope-manipulation, dynamic-manipulation, one-shot, sim-to-real, heterogeneous-system, soft-tool-use]
    status: weakly_supported
    confidence: 0.65
    domain: Robotics
  - slug: heterogeneous-payload-rope-dynamics-implicit-vs
    title: "Heterogeneous payload-on-rope dynamics are captured implicitly without per-trial residual updates, in contrast to IRP's iterative residual loop"
    tags: [implicit-system-identification, residual-physics, iterative-residual-policy, heterogeneous-system, dynamic-manipulation, one-shot]
    status: weakly_supported
    confidence: 0.55
    domain: Robotics
  - slug: task-level-ilc-real-hardware-flying-knot-100pct-under-10-trials
    title: "Task-Level ILC from one human demo plus a 5-parameter rope model achieves 100% success in fewer than 10 real-hardware trials of the flying knot across 7 rope and cable types"
    tags: [ILC, iterative-learning-control, deformable-manipulation, rope, real-world-learning, sample-efficiency, xArm-7, flying-knot]
    status: supported
    confidence: 0.75
    domain: Robotics
  - slug: task-level-ilc-cross-rope-transfer-2-to-5-trials
    title: "Cross-rope command transfer with Task-Level ILC requires only 2-5 additional real trials for most rope-pair source/targets"
    tags: [ILC, iterative-learning-control, deformable-manipulation, rope, transfer-learning, real-world-learning, sample-efficiency]
    status: weakly_supported
    confidence: 0.65
    domain: Robotics
  - slug: apg-dominates-shac-and-ppo-on-whiprope-low-level
    title: "DaXBench Whip-Rope baselines: APG 0.83 dominates SHAC 0.66 and PPO 0.25 — analytic policy gradients win on dynamic deformable tasks"
    tags: [DLO, DOM, deformable-object-manipulation, RL, differentiable-physics, APG, SHAC, PPO, benchmark, low-level-control]
    status: supported
    confidence: 0.7
    domain: Robotics
  - slug: jax-differentiable-rope-enables-batched-rl-vs-cem-mpc
    title: "JAX-based differentiable rope simulators enable batched RL training that gradient-free baselines (CEM-MPC) cannot match on dynamic rope tasks"
    tags: [DLO, DOM, JAX, differentiable-physics, MPM, batched-RL, CEM-MPC, APG, SHAC, benchmark]
    status: weakly_supported
    confidence: 0.55
  - slug: wiggle-sysid-enables-zero-shot-3d-rope-striking
    title: "A task-agnostic 'wiggle' observation predicts rope physical parameters well enough to enable zero-shot real-hardware 3D-point-striking via downstream CMA-ES trajectory optimization"
    tags: [DLO, rope-manipulation, system-identification, zero-shot, sim-to-real, dynamic-manipulation, drake, xarm7]
    status: supported
    confidence: 0.65
    domain: Robotics
  - slug: decoupled-sysid-beats-iterative-residual-on-zero-retry
    title: "Decoupling sysID from task execution beats end-to-end iterative residual policies on the 3D rope-tip striking task at deployment time"
    tags: [DLO, rope-manipulation, system-identification, iterative-residual-policy, dynamic-manipulation, zero-shot]
    status: weakly_supported
    confidence: 0.5
    domain: Robotics
  - slug: dynamic-cable-apex-point-arc-policy
    title: "A learned 3D apex point that parameterizes a minimum-jerk QP trajectory is sufficient to learn target-conditioned dynamic cable behaviors (vaulting / knocking / weaving) on real hardware across multiple cables"
    tags: [DLO, dynamic-manipulation, apex-point, self-supervised, real-robot, behavior-cloning]
    status: supported
    confidence: 0.7
    domain: Robotics

Summary:

foundations:
  - slug: backpropagation
    title: "Backpropagation"
    status: mainstream
    domain: general
  - slug: behavioral-cloning
    title: "Behavioral Cloning"
    status: mainstream
    domain: Robotics
  - slug: contact-rich-manipulation
    title: "Contact-Rich Manipulation"
    status: mainstream
    domain: Robotics
  - slug: cosserat-rod-theory
    title: "Cosserat Rod Theory"
    status: mainstream
    domain: Robotics
  - slug: cross-validation
    title: "Cross-Validation"
    status: mainstream
    domain: general
  - slug: deformable-linear-object
    title: "Deformable Linear Object"
    status: mainstream
    domain: Robotics
  - slug: diffusion-policy
    title: "Diffusion Policy"
    status: mainstream
    domain: Robotics
  - slug: discrete-elastic-rods
    title: "Discrete Elastic Rods"
    status: mainstream
    domain: Robotics
  - slug: domain-randomization
    title: "Domain Randomization"
    status: mainstream
    domain: Robotics
  - slug: finite-element-method
    title: "Finite Element Method"
    status: mainstream
    domain: Robotics
  - slug: forward-kinematics
    title: "Forward Kinematics"
    status: mainstream
    domain: Robotics
  - slug: gradient-descent
    title: "Gradient Descent"
    status: mainstream
    domain: general
  - slug: grasping
    title: "Grasping"
    status: mainstream
    domain: Robotics
  - slug: imitation-learning
    title: "Imitation Learning"
    status: mainstream
    domain: Robotics
  - slug: impedance-control
    title: "Impedance Control"
    status: mainstream
    domain: Robotics
  - slug: inverse-kinematics
    title: "Inverse Kinematics"
    status: mainstream
    domain: Robotics
  - slug: jacobian-based-control
    title: "Jacobian-Based Control"
    status: mainstream
    domain: Robotics
  - slug: mass-spring-system
    title: "Mass-Spring System"
    status: mainstream
    domain: Robotics
  - slug: model-based-reinforcement-learning
    title: "Model-Based Reinforcement Learning"
    status: mainstream
    domain: Robotics
  - slug: model-predictive-control
    title: "Model Predictive Control"
    status: mainstream
    domain: Robotics
  - slug: optimization
    title: "Optimization"
    status: mainstream
    domain: general
  - slug: position-based-dynamics
    title: "Position-Based Dynamics"
    status: mainstream
    domain: Robotics
  - slug: regularization
    title: "Regularization"
    status: mainstream
    domain: general
  - slug: shape-servoing
    title: "Shape Servoing"
    status: mainstream
    domain: Robotics
  - slug: sim-to-real-transfer
    title: "Sim-to-Real Transfer"
    status: mainstream
    domain: Robotics
  - slug: tactile-sensing
    title: "Tactile Sensing"
    status: mainstream
    domain: Robotics
  - slug: visual-servoing
    title: "Visual Servoing"
    status: mainstream
    domain: Robotics
  - slug: visuomotor-policy
    title: "Visuomotor Policy"
    status: mainstream
    domain: Robotics
