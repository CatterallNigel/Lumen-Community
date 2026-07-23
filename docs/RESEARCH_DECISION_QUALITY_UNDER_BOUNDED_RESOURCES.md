# Decision Quality Under Bounded Resources (DQBR)

**Status:** Research

---

## Abstract

Current AI benchmarks primarily measure model capability. They generally do not measure whether an AI system selected an appropriate course of action given the resources available to it.

This document proposes **Decision Quality Under Bounded Resources (DQBR)** as a research direction for evaluating AI systems based upon the quality of their decisions rather than raw model capability.

DQBR is currently a conceptual framework and should not be considered a formal metric or benchmark.

This document proposes a research direction rather than a completed methodology. No accepted definition, scale, benchmark, or implementation currently exists. Its purpose is to preserve the concept, define its motivation, and establish a foundation for future investigation.

---

## Scope

DQBR is **not** intended to replace existing model benchmarks.

Benchmarks such as MMLU, HumanEval and SWE-Bench primarily measure aspects of model capability under defined evaluation conditions.

DQBR instead proposes evaluating the quality of decisions made by an AI system while operating under bounded resources.

The framework is intended to complement existing capability benchmarks rather than replace them.

---

## Terminology

### Decision Quality

The appropriateness of an AI system's selected course of action relative to its objective and the resources available at the time the decision was made.

### Bounded Resources

The finite computational, informational, temporal and operational resources available to an AI system while solving a task.

---

## Response Evaluation

The quality of an AI response cannot be represented by a single measure.

This document proposes three complementary observations.

### Satisfaction

Establishes whether the response satisfied the explicit user request.

### Enrichment

Establishes whether the response transferred useful understanding beyond the minimum necessary to satisfy the request.

### Capability Utilization

Establishes whether the response expressed the fullest understanding the model had developed during task execution, given its available knowledge and reasoning.

These measures are complementary rather than hierarchical. A response may satisfy the user's ask while providing limited enrichment, or may demonstrate significant enrichment while failing to express the model's full developed understanding.

From an evaluation perspective:

* **Satisfaction** is directly observable.
* **Enrichment** is comparatively observable.
* **Capability Utilization** is inferential.

---

## Motivation

Modern AI systems operate under constraints.

These include, but are not limited to:

* Context window
* Latency
* Available memory
* Compute
* Energy
* API cost
* Hardware capability
* Available tools
* Available information
* User objectives

Current benchmarks typically ignore these constraints.

Real-world systems cannot.

---

## Inspiration

The work draws inspiration from Herbert Simon's concepts of **Bounded Rationality** and **Satisficing**.

Rather than assuming perfect optimisation, Simon argued that intelligent agents make satisfactory decisions within finite resources.

DQBR extends this idea beyond the acceptance of a satisfactory outcome to the broader evaluation of decision quality within finite computational, informational and operational constraints.

---

## Current AI Evaluation

Current benchmark suites primarily measure model capability.

Examples include:

* MMLU
* HumanEval
* SWE-Bench

These benchmarks provide valuable measures of knowledge, reasoning and task performance.

However, they generally do not evaluate:

* whether an appropriate model was selected;
* whether the available tools were used effectively;
* whether retrieval depth was appropriate;
* whether additional reasoning would have been beneficial;
* whether resources were allocated efficiently;
* whether the system stopped at an appropriate point.

DQBR proposes that these decisions are themselves worthy of evaluation.

---

## Proposed Concept

Traditional evaluation asks:

> **Can the model solve the task?**

DQBR instead asks:

> **Did the AI system choose an appropriate course of action given the objective and the resources available?**

The emphasis shifts from evaluating a single model toward evaluating the behaviour of the complete AI system.

---

## Decision

Within DQBR, a **decision** is any selection made by an AI system that influences the subsequent execution of a task.

Examples include:

* Model selection
* Tool selection
* Retrieval strategy
* Context allocation
* Reasoning depth
* Verification strategy
* Stopping decisions
* Response generation strategy

DQBR proposes evaluating the quality of these decisions relative to the resources available at the time they were made.

Importantly, DQBR is concerned with the **sequence of decisions** made during task execution rather than a single isolated decision.

---

## Resources

Examples of bounded resources include:

* Context window
* Time
* Latency
* Token budget
* Hardware capability
* Compute
* Energy
* Cost
* Available tools
* Available knowledge
* Human interaction
* Memory

The relative importance of these resources will vary according to the task.

---

## Decision Variables

Possible variables influencing decision quality include:

* Model selection
* Tool selection
* Retrieval depth
* Reasoning depth
* Context utilisation
* Information gathering
* Confidence estimation
* Uncertainty management
* Verification strategy
* Stopping point

These variables may interact and should not necessarily be considered independently.

---

## Example

Consider identical programming tasks executed under different resource constraints.

A local laptop may favour a compact local model.

A high-performance server may justify using a much larger model.

Although the objective is identical, the highest-quality decision may differ because the available resources differ.

DQBR proposes evaluating the appropriateness of these decisions rather than simply comparing the capability of the underlying models.

---

## Relationship to Satisficing

Satisficing should not be interpreted as merely producing a "good enough" answer.

Within Herbert Simon's work, satisficing represents an intelligent stopping rule under bounded rationality.

Modern AI systems continually make comparable decisions, including:

* when to retrieve additional information;
* when to invoke external tools;
* when to continue reasoning;
* when to verify an answer;
* when additional computation is unlikely to improve the outcome.

DQBR proposes that the quality of these decisions forms an important part of evaluating AI systems.

---

## Potential Research Questions

* Can DQBR become objectively measurable?
* Can decision quality be separated from model capability?
* Can orchestration improve DQBR independently of model size?
* Can smaller models outperform larger models under constrained resources?
* Can DQBR assist automatic model selection?
* Can DQBR assist tool selection?
* Can DQBR evaluate orchestration systems independently of the underlying models?
* Can response evaluation (Satisfaction, Enrichment and Capability Utilization) be related quantitatively to decision quality?

---

## Challenges

Several significant challenges remain.

* No accepted scale currently exists.
* Decision quality is inherently multi-dimensional.
* Ground truth may be difficult to establish.
* Human satisfaction is subjective.
* Capability utilization cannot be directly observed.
* Resource costs vary between environments.
* The framework may require entirely new benchmark methodologies.

These challenges represent opportunities for future research rather than deficiencies in the concept.

---

## Future Work

This document captures an early research direction.

The framework should evolve through experimentation, discussion and empirical evaluation.

Future work may include:

* formal terminology;
* measurable metrics;
* experimental methodologies;
* benchmark design;
* orchestration evaluation;
* empirical validation.

No formal implementation currently exists.
