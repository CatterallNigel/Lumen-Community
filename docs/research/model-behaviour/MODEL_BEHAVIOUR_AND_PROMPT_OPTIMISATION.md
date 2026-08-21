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

Checkpoint analysis should always be interpreted within the context of a Project and its active Knowledge Branch.

Successive cognitive distillations provide an observable record of how a model's understanding changes over time.

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

# Cognitive Checkpoint Design Principles

The purpose of a Cognitive Checkpoint is to preserve the model's current state of understanding so that reasoning can continue naturally after context loss.

A checkpoint is not intended to produce a final architectural assessment, nor to predict future understanding.

Its role is to record what the model currently knows.

## Present understanding rather than future understanding

Experimental evaluation demonstrated that a Cognitive Checkpoint should preserve the model's present state of understanding, not require it to predict its future state of understanding.

Questions that require knowledge of information not yet observed should be avoided.

For example, asking a model to identify future architectural pressure, unresolved questions, or missing information while it is still incrementally reading a source file may require reasoning beyond its current knowledge.

Such questions encourage prediction rather than observation.

## The jigsaw principle

An incremental checkpoint is analogous to constructing a jigsaw puzzle.

After connecting only a small number of pieces, it is impossible to determine which remaining pieces are missing because the complete picture is not yet known.

Similarly, an LLM reading a source file incrementally cannot reliably determine what architectural concepts, unresolved questions, or refactoring opportunities may emerge from source code it has not yet read.

The checkpoint should therefore describe the picture currently assembled rather than speculate about the unseen portions.

## Record observations, not predictions

Checkpoint prompts should ask only questions that can be answered from the model's present cognitive state.

Questions requiring future knowledge should not form part of incremental checkpoint generation.

Suitable checkpoint content includes:

- Established understanding.
- New understanding acquired since the previous checkpoint.
- Observable evidence supporting that understanding.
- Current architectural model.
- Confidence based upon observed source material.

Checkpoint prompts should avoid requiring predictions about information that cannot yet exist within the model's current state of knowledge.

## Small prompt changes may produce large behavioural changes

Experiments comparing Lumen v3.2.6 and v3.2.7 demonstrated that relatively small changes to a checkpoint prompt produced disproportionately large changes in the model's observable reasoning behaviour.

Although only a single checkpoint section was modified, the resulting cognitive distillations became significantly more conservative and preserved less architectural understanding.

Prompt modifications should therefore be treated as experimental engineering changes rather than editorial refinements.

Only one checkpoint hypothesis should be evaluated at a time, using identical models, objectives, input material, and benchmark tasks wherever practical.

## Experimental findings

Current evidence suggests:

- Richer checkpoints are produced when the model is asked to record its current understanding.
- Requiring the model to predict future understanding can reduce checkpoint quality.
- Empty checkpoint sections may be behaviourally neutral, whereas speculative checkpoint sections may alter reasoning behaviour.
- Checkpoint quality should be evaluated by the observable evolution of understanding rather than by the number of completed sections.

These findings remain subject to further validation as additional checkpoint experiments are performed.

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
