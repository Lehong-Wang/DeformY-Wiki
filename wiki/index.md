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
