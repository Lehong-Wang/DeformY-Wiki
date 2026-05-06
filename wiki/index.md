# Wiki Index

papers:
  - slug: deformx-versatile-co-simulation-framework-deformable
    title: "DeformX: A Versatile Co-Simulation Framework for Deformable Linear Objects"
    tags: [DLO, deformable-linear-object, cosserat-rod, simulation, sim-to-real, robot-learning, dataset, isaac-sim]
    importance: 3
    domain: Robotics
  - slug: daxbench-benchmarking-deformable-object-manipulation-differentiable
    title: "DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics"
    tags: [DLO, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning, APG, SHAC, PPO, sim-to-real]
    importance: 4
    domain: Robotics

concepts:
  - slug: cosserat-isaac-cosimulation
    title: "Cosserat-Isaac Co-Simulation"
    tags: [DLO, simulation, cosserat-rod, isaac-sim, sim-to-real, robot-learning]
    maturity: emerging
  - slug: differentiable-deformable-benchmark
    title: "Differentiable Deformable-Object Benchmark"
    tags: [DLO, DOM, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning]
    maturity: emerging

topics:

people:
  - slug: siwei-chen
    name: "Siwei Chen"
    affiliation: "National University of Singapore (AdaComp Lab)"
    tags: [deformable-object-manipulation, differentiable-physics, JAX, MPM, RL, robot-learning, benchmark]

ideas:

experiments:

claims:
  - slug: cosserat-physics-narrows-dlo-swinging-sim2real
    title: "Cosserat physics narrows the DLO swinging sim-to-real gap"
    tags: [DLO, sim-to-real, cosserat-rod, robot-learning, PPO]
    status: weakly_supported
    confidence: 0.55
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
