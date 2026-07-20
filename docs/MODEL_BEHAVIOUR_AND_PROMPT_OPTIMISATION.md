# Model Behaviour & Prompt Optimisation

**Status:** Living Research Document

---

# Purpose

This document defines Lumen's approach to understanding the observable behaviour of Large Language Models (LLMs) and using that understanding to develop evidence-based prompt engineering techniques.

The objective is not simply to produce better prompts, but to understand *why* particular prompts, orchestration policies, and workflows produce different outcomes.

Lumen treats prompt engineering as an engineering discipline that should be based upon observation, experimentation, repeatability, and evidence.

---

# Philosophy

Prompt engineering is often performed through trial and error.

Lumen seeks to replace intuition with measurement.

Rather than concentrating solely on a model's final response, Lumen observes how the model's reported understanding evolves throughout a task.

This observable cognitive evolution becomes the basis for experimentation, analysis, and continual improvement.

---

# Scope

This research applies to:

* Prompt design
* System prompts
* Checkpoint prompts
* Distillation prompts
* Resume prompts
* Orchestration policies
* Context management
* Session continuity
* Model comparison
* Prompt optimisation

---

# Guiding Principles

* Measure before optimising.
* Preserve evidence.
* Compare observations rather than assumptions.
* Treat prompts as versioned engineering artefacts.
* Prefer repeatable experiments over anecdotal success.
* Separate observation from interpretation.
* Keep all conclusions evidence-based.

---

# Observable Cognitive State

Lumen does not attempt to expose or infer a model's internal reasoning.

Instead, it captures and analyses the model's externally observable understanding through successive cognitive distillations.

This distinction is important.

Lumen studies the evolution of a model's reported understanding rather than attempting to reconstruct its private reasoning process.

---

# Research Objectives

The programme seeks to understand questions including:

* How does the model prioritise competing instructions?
* Under what conditions are rules strengthened or weakened?
* What constitutes a contradiction from the model's perspective?
* How are conflicting instructions resolved?
* How stable are architectural decisions across long sessions?
* How effectively is continuity preserved?
* How much context is required for reliable continuation?
* Which prompt structures produce the highest quality cognitive distillations?
* Which prompt patterns minimise cognitive drift?
* How consistent are different models when presented with identical tasks?

---

# Experimental Methodology

Each experiment should define:

* Objective
* Model
* Model version
* Prompt version
* Task
* Input material
* Context size
* Distillation sequence
* Final response
* Human assessment
* External reviewer assessment
* Conclusions

Experiments should be repeatable wherever practical.

---

# Checkpoint Analysis

Successive cognitive distillations provide an observable record of how a model's understanding changes over time.

Areas of interest include:

* New understanding
* Lost understanding
* Stable understanding
* Contradictions
* Assumptions
* Corrections
* Objective drift
* Provenance retention
* Continuity quality

---

# Prompt Optimisation

Prompt optimisation should always be driven by evidence gathered through experimentation.

Evaluation may consider:

* Accuracy
* Consistency
* Stability
* Continuity preservation
* Architectural correctness
* Provenance retention
* Hallucination resistance
* Recovery after session rollover
* Distillation quality

---

# External Review

Independent review by external models provides an additional source of analysis.

These reviewers do not replace human judgement.

Instead, they provide another observable data point that can be compared against:

* Raw cognitive distillations
* Human assessment
* Previous experiments

The purpose is to evaluate both the model being studied and the quality of the review process itself.

---

# Expected Outcomes

Over time this research should produce:

* A Prompt Optimisation Guide.
* Model-specific engineering guidance.
* Comparative behaviour studies across multiple models.
* Best practices for continuity preservation.
* Improved checkpoint and distillation prompts.
* Better orchestration policies.
* Evidence-based prompt design patterns.

---

# Long-Term Vision

Lumen aims to make prompt engineering an evidence-based engineering discipline.

By observing, measuring, comparing, and validating model behaviour over time, prompt optimisation becomes a repeatable process rather than an exercise in intuition.

The resulting knowledge benefits both Lumen itself and the wider community by improving understanding of how modern language models maintain, evolve, and preserve their observable cognitive state.
