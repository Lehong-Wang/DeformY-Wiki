---
title: "Meta-Learning"
slug: "meta-learning"
domain: "general"
status: mainstream
aliases: ["learning to learn", "meta learning", "few-shot learning"]
first_introduced: "Schmidhuber 1987; Thrun & Pratt 1998; Finn et al. 2017 (MAML)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Meta-learning_(computer_science)"
---

## Definition

Meta-learning, or "learning to learn," is a subfield of machine learning in which a learning procedure is itself improved across a distribution of tasks, so that the model can adapt rapidly to a new task from very little data. Rather than fitting a single task, a meta-learner is trained on many tasks drawn from a task distribution and optimizes for the ability to generalize and adapt, not just to memorize.

## Intuition

A model trained the ordinary way is good at the data it saw; a meta-learned model is good at *getting* good quickly. The classic setup splits each task into a small support set (used to adapt) and a query set (used to score the adaptation). By optimizing query performance after adaptation, across thousands of tasks, the system learns an inductive bias — a weight initialization, a learned optimizer, or a fast inference mechanism — that makes new tasks easy. For robotics, "task" is often a new dynamics regime (payload, friction, terrain), and fast adaptation means recovering control after the world changes.

## Formal notation (LLM analysis)

Given a task distribution $p(\mathcal{T})$, where each task $\mathcal{T}_i$ has support data $\mathcal{D}_i^{\text{tr}}$ and query data $\mathcal{D}_i^{\text{val}}$, meta-learning solves
$$\min_\theta \ \mathbb{E}_{\mathcal{T}_i \sim p(\mathcal{T})} \big[\, \mathcal{L}_{\mathcal{D}_i^{\text{val}}}\big(\mathcal{A}(\theta, \mathcal{D}_i^{\text{tr}})\big) \,\big],$$
where $\mathcal{A}$ is the adaptation operator. In gradient-based meta-learning (MAML), $\mathcal{A}(\theta, \mathcal{D}) = \theta - \alpha \nabla_\theta \mathcal{L}_{\mathcal{D}}(\theta)$, so the outer objective differentiates through one or more inner gradient steps. In amortized/context-based meta-learning, $\mathcal{A}$ infers a task embedding $z = q_\phi(\mathcal{D}^{\text{tr}})$ that conditions the predictor.

## Key variants

- **Gradient-based (optimization) meta-learning** — MAML, Reptile, FOMAML: learn an initialization (or learning rate) from which a few gradient steps adapt to a new task.
- **Metric/embedding-based** — Prototypical/Matching/Relation Networks: learn an embedding where nearest-neighbor classification solves few-shot tasks.
- **Context/amortized (black-box) meta-learning** — RNN or attention models that infer a task embedding from context and condition predictions (the basis of context-based meta-RL and RMA-style adaptation modules).
- **Meta-learned dynamics models** — forward models conditioned on a fast-inferred latent that captures the current physical regime (e.g. Learning-to-Adapt).

## Known limitations

Meta-training is expensive and the result only generalizes within the training task distribution — out-of-distribution tasks adapt poorly. Gradient-based variants involve costly second-order terms (often approximated). Performance is sensitive to the task-distribution design and to the support-set size. Defining a meaningful, diverse task distribution for real-world robotics is itself hard.

## Open problems (LLM analysis)

Robust adaptation under distribution shift beyond the meta-training tasks; meta-learning with realistic, non-i.i.d. task streams; unifying gradient-based and amortized adaptation; and calibrated uncertainty about when a newly inferred task embedding is trustworthy.

## Relevance to active research (LLM analysis)

Meta-learning is the conceptual backbone of online adaptation in robot learning: Learning-to-Adapt meta-trains a dynamics model for fast regime inference, and RMA's adaptation module is an amortized inference of latent environment factors. Both are instances of "learn to adapt fast," making this foundation a direct prerequisite for the ingested method papers.
