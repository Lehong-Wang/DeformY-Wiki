---
title: "Teacher-Student Learning"
slug: "teacher-student-learning"
domain: "general"
status: mainstream
aliases: ["privileged learning", "privileged-information distillation", "teacher-student distillation", "policy distillation"]
first_introduced: "Vapnik & Vashist 2009 (learning using privileged information); Hinton et al. 2015 (distillation)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Knowledge_distillation"
---

## Definition

Teacher-student learning is a privileged-information distillation paradigm: a high-capacity or privileged *teacher* policy/model is trained with access to information unavailable at deployment (e.g. ground-truth simulator state, future observations, or a large pretrained network), and its behavior is then distilled into a deployable *student* that relies only on the observations available at runtime (onboard sensors and observation history). It generalizes knowledge distillation — transferring competence from a large/privileged model to a smaller/restricted one without loss of validity — and is the mechanism behind RMA and many sim-to-real pipelines.

## Intuition

Some information is cheap in simulation but impossible to measure on a real robot — exact friction, mass, terrain height, contact forces. A teacher trained with this privileged state can learn an excellent policy easily. The student cannot see those quantities, but it *can* learn to imitate the teacher's actions using only what it will actually have at test time, often inferring the hidden factors implicitly from a short history of proprioception. The result is a deployable controller that behaves as if it knew the privileged state, because it was supervised by something that did.

## Formal notation (LLM analysis)

Train a teacher $\pi_T(a \mid s, e)$ on full state $s$ plus privileged context $e$ (e.g. by RL in simulation). Then train a student $\pi_S(a \mid o_{t-k:t})$ from on-board observation history to match the teacher via supervised imitation:
$$\min_{\theta_S}\ \mathbb{E}_{(s,e)\sim d_{\pi_T}}\big[\, D\big(\pi_T(\cdot\mid s, e),\ \pi_S(\cdot\mid o_{t-k:t})\big) \,\big],$$
where $D$ is a behavior-matching loss (e.g. action MSE, KL of action distributions, or DAgger-style on-policy correction). In RMA specifically, a teacher encodes privileged environment factors into a latent $z_t = \mu(e_t)$, and an adaptation module learns to regress $\hat z_t$ from observation history so the same base policy runs at deployment.

## Key variants (LLM analysis)

- **Knowledge distillation** — soft-label transfer from a large teacher to a smaller student.
- **Learning using privileged information (LUPI)** — teacher exploits side information available only at training time.
- **Privileged-to-onboard distillation (sim-to-real)** — teacher uses simulator ground truth; student uses sensors/history (RMA, "Learning by Cheating").
- **DAgger-style on-policy distillation** — student queries the teacher on its own visited states to fight distribution shift.
- **Latent-factor regression (RMA adaptation module)** — student infers a privileged latent from history rather than copying actions directly.

## Known limitations (LLM analysis)

The student can only succeed if the privileged information is recoverable (or made irrelevant) from its available observations; otherwise it caps below the teacher. Naive offline behavior cloning suffers compounding distribution shift, motivating on-policy correction. The teacher's quality bounds the student, and any sim-to-real gap in the teacher's training environment propagates to the student.

## Open problems (LLM analysis)

Guaranteeing the student matches teacher performance when privileged factors are only partially observable from history; minimizing demonstrations/queries for on-policy distillation; distilling across large embodiment or modality gaps; and certifying that the distilled student is safe outside the teacher's training distribution.

## Relevance to active research (LLM analysis)

Teacher-student learning is the precise mechanism behind RMA: a privileged teacher and base policy are trained in simulation, then an adaptation module is distilled to infer environment factors from on-board history for real-world deployment. The same recipe underlies broad sim-to-real and legged-locomotion work, so this foundation is a direct prerequisite for the RMA paper being ingested.
