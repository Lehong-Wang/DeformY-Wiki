---
name: "PETS: Probabilistic Ensembles with Trajectory Sampling"
slug: "pets-probabilistic-ensembles-trajectory-sampling"
type: inference
tags: [model-based-reinforcement-learning, probabilistic-ensemble, trajectory-sampling, uncertainty-propagation, model-predictive-control, cross-entropy-method, sampling-based-planning, sample-efficiency]
source_papers: ["[[deep-reinforcement-learning-handful-trials-using]]"]
parent_methods: []
child_methods: []
realizes_concepts: ["[[trajectory-sampling-uncertainty-propagation]]"]
code_repo: "https://github.com/kchua/handful-of-trials"
date_updated: 2026-06-16
---

## Problem setting

A model-based RL agent must control a continuous, possibly contact-rich system from few interactions, by learning a forward dynamics model and planning with it. The central failure mode is that a high-capacity neural dynamics model **overfits in the low-data regime** and a planner then **exploits the model's errors**, so MBRL historically converged to worse asymptotic returns than model-free RL. PETS targets agents that (a) can fit a deep dynamics model on the data collected so far, (b) have a known reward/cost to score candidate trajectories, and (c) can afford sampling-based MPC at control time. It assumes nothing about smoothness of the dynamics (unlike GP-MBRL) and learns no policy network.

## Mechanism

PETS combines three components, each chosen by ablation:

- **Probabilistic ensemble (PE) dynamics model.** $B$ bootstrapped neural networks, each *probabilistic* — outputting a Gaussian $\mathcal{N}(\mu_\theta(s,a),\Sigma_\theta(s,a))$ with input-dependent diagonal covariance, trained by negative log-likelihood. Probabilistic outputs capture **aleatoric** uncertainty (inherent stochasticity); ensemble disagreement across bootstraps captures **epistemic** uncertainty (finite-data subjectivity). PE is the only model among {Deterministic, Probabilistic, Deterministic-Ensemble, GP, PE} that captures both; measured ranking PE > P > DE > D. ($B=5$ sufficed for all tasks.)
- **Trajectory sampling (TS) propagation.** To evaluate an action sequence under the PE model, propagate $P$ particles, each stepped by one bootstrap network sampled per its $\Sigma$. **TS1** re-draws each particle's bootstrap every step (marginal-posterior sampling); **TS∞** fixes each particle's bootstrap for the whole trial, which makes aleatoric and epistemic state variance *separable* (aleatoric = mean within-bootstrap variance; epistemic = variance of per-bootstrap means). ($P=20$.)
- **CEM-based MPC planning.** Receding-horizon control: each step, optimize the action sequence $\arg\max_{a_{t:t+H}}\sum_\tau \tfrac{1}{P}\sum_p r(s^p_\tau,a_\tau)$ with the **cross-entropy method** (iteratively refit a sampling distribution to the elite action samples — beats random shooting), execute only the first action, replan next step.

The distinctive, reusable design is the *pairing*: an ensemble-of-probabilistic-nets dynamics model with a particle propagation scheme that preserves (and separates) its two uncertainty types, feeding a gradient-free sampling planner that is thereby robust to model error and over-long horizons.

## Procedure

1. **Initialize** dataset $\mathcal{D}$ with one trial under a random controller.
2. **For each trial** $k=1,\dots,K$:
   1. **Train** the PE dynamics model $\tilde f$ on $\mathcal{D}$ (each of the $B$ nets on its own bootstrap resample, NLL loss; bound the predicted variance to avoid arbitrary OOD variance disrupting planning).
   2. **For each timestep** $t$ up to the task horizon:
      - Run CEM: sample candidate action sequences $a_{t:t+H}\sim \text{CEM}(\cdot)$; for each, propagate $P$ particles with **TS** through $\tilde f$; score by mean per-particle cumulative reward; update the CEM sampling distribution toward the elites; iterate (≈5 CEM iterations × ~500 candidates).
      - **Execute** only the first optimized action $a^*_t$.
      - **Append** the observed transition $(s_t,a^*_t,s_{t+1})$ to $\mathcal{D}$.

Representative hyperparameters: $B=5$ bootstrap nets, $P=20$ particles, CEM 5 iterations × 500 candidates; per-timestep replanning; MPC horizon $H$ chosen per task (probabilistic propagation tolerates an over-long $H$).

## Assumptions

- A **known reward/cost** function to score sampled trajectories at plan time.
- The system is **Markov** in the observed state, and a forward dynamics model is learnable from collected transitions.
- **Bootstrap disagreement is an acceptable epistemic proxy** (cheaper than full Bayesian NN inference) — reasonable, but not a calibrated posterior.
- Aleatoric noise is adequately captured by an input-dependent **Gaussian** (unimodal per step); multimodality across trajectories is recovered by the particle set.
- Sampling-based MPC at every timestep is **computationally affordable** for the control rate.

## Limitations

- **No amortized policy** — per-step CEM-MPC is expensive; the original work could not backprop an effective policy through the uncertainty-aware model (suspected chaotic policy gradients).
- **Epistemic uncertainty is separated but not exploited** for directed exploration in the original algorithm (left to future work).
- **OOD variance** of probabilistic nets is arbitrary and must be explicitly bounded for stable planning.
- **Ensemble + particle cost** ($B{\times}P{\times}$ CEM samples) in compute and memory vs. a single deterministic model.
- Inherits residual **model-based sub-optimality**: matches model-free asymptotes on the studied benchmarks, but the gap is bridged in part, not in general.

## Tradeoff profile

- **Sample efficiency:** very high — e.g. ~8× fewer samples than SAC and ~125× fewer than PPO on half-cheetah to reach comparable returns.
- **Asymptotic performance:** matches model-free PPO/SAC on the benchmark tasks (the result that defined the method).
- **Robustness to model error:** high — particle separation makes the planner ignore the unpredictable far future, so MPC tolerates over-long horizons where deterministic propagation compounds bias.
- **Compute at control time:** high — sampling-based MPC + ensemble + particles every step; no policy to amortize planning.
- **Generality:** broad — model-agnostic across continuous-control tasks; no smoothness assumption (unlike GP-MBRL); the dominant lever is uncertainty at *learning* time, with propagation choice secondary given a good PE model.

## Evaluated by

- [[deep-reinforcement-learning-handful-trials-using]] — Cartpole, 7-DoF Pusher, 7-DoF Reacher, and Half-cheetah (MuJoCo): matches PPO/SAC asymptotes in <100 trials, with 8-125× sample reductions; ablations establish the PE > P > DE > D model ranking and that uncertainty at learning time is the decisive component.
