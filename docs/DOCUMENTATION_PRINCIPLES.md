# Documentation Principles

Documentation is treated as an engineering artefact.

Its purpose is to describe Lumen accurately, consistently and unambiguously. Every document should help readers understand the architecture, behaviour and design decisions of the system without relying on marketing language or unsupported claims.

The documentation should remain valuable throughout the lifetime of the project, evolving alongside the implementation while maintaining historical accuracy.

---

# Documentation Principles

## Precision over persuasion

Documentation should explain what Lumen does rather than attempt to persuade readers of its value.

Technical accuracy is always preferred over promotional language.

---

## Distinguish present from future

Implemented capabilities, planned capabilities and long-term architectural direction must be clearly distinguished.

Documentation must never imply that a planned feature already exists.

---

## Evidence-based descriptions

Where architectural or technical claims are made they should be supported by explanation or evidence wherever practical.

Avoid vague or absolute statements.

---

## Engineering first

Documentation is written primarily for engineers, architects and technically minded readers.

Concepts should be explained clearly without sacrificing technical correctness.

---

## Consistency

Terminology should be used consistently throughout the project.

Where a technical term has a specific meaning within Lumen, that meaning should remain consistent across all documentation.

---

## Model independence

Documentation should avoid describing behaviour in terms of a particular language model unless discussing model-specific behaviour.

Lumen is designed to remain independent of any individual client, model or provider.

---

## Honest limitations

Limitations are documented openly.

Known constraints, assumptions and areas under development should be described rather than hidden.

---

## Stable concepts

Documentation should describe enduring architectural principles rather than implementation details wherever practical.

For example, prefer:

> Lumen preserves continuity across context boundaries.

rather than:

> Lumen solves 32K context windows.

The first remains true regardless of future model capabilities.

---

## Preferred language

Prefer words such as:

- preserves
- orchestrates
- maintains
- records
- validates
- documents
- explains
- enables

Avoid marketing language such as:

- revolutionary
- game changing
- next generation
- cutting edge
- AI-powered

---

## Documentation quality

Documentation is reviewed with the same care as source code.

Clarity, correctness and consistency are considered engineering responsibilities.

---

# Dictionary of Terms

This section defines the terminology used throughout the Lumen project.

## Continuity

The ability to preserve and restore the knowledge required for work to continue consistently across sessions, context boundaries and time.

Continuity is the primary problem Lumen addresses.

---

## Context Boundary

The point at which a language model can no longer retain all previous conversational information because of finite context capacity.

Lumen preserves continuity across context boundaries.

---

## Project

The persistent unit of work being managed by Lumen.

A project represents a body of work, such as a software system, research effort, engineering task or other long-running activity.

A project may span multiple sessions, models, clients and days of work.

---

## Lumen Project

The software project that develops Lumen itself.

Unless explicitly stated otherwise, the term *project* throughout the Lumen documentation refers to a project being managed by Lumen rather than the development of Lumen itself.

---

## Session

A single interaction with an underlying language model.

Sessions are transient.

Projects persist beyond sessions.

---

## Objective

The purpose or goal that the project is attempting to achieve.

Objectives should remain stable throughout the lifetime of the project unless intentionally revised.

---

## Checkpoint

A recorded representation of the project's current continuity state at a particular point in time.

Checkpoints enable continuity restoration.

---

## Distillation

A structured summary produced to preserve the information necessary to continue work while reducing context consumption.

A distillation preserves continuity rather than replacing original source material.

---

## Provenance

The origin of information.

Provenance records where conclusions, facts or decisions originated.

---

## Continuity State

The complete collection of information required to resume a project consistently.

This may include objectives, checkpoints, reasoning, provenance, assumptions, unresolved questions and other project knowledge.

---

## Project Knowledge

The accumulated knowledge describing the current state of a project.

Examples include objectives, established facts, architectural decisions, reasoning, provenance and continuity artefacts.

Project knowledge persists independently of any individual language model session.

---

## Continuity Artefact

Any persisted item that contributes to preserving project continuity.

Examples include checkpoints, distillations, objectives, provenance records, architectural decisions, assumptions and other recorded project knowledge.

Continuity artefacts collectively form the continuity state required to resume work consistently.

---

## Orchestration

The coordination of clients, language models, tools and continuity processes required to complete a task.

Lumen orchestrates workflows rather than performing model inference itself.

---

## Trustworthiness

The degree to which conclusions can be understood, traced, reproduced and justified through preserved evidence and provenance.

---

## Auditability

The ability to examine how a conclusion or decision was reached through recorded reasoning, provenance and continuity artefacts.

---

## Model Independence

The architectural principle that Lumen should operate with different language models without depending upon model-specific behaviour wherever practical.

---

## Client Independence

The architectural principle that Lumen should operate independently of any particular user interface or development environment.

---

## Engineering Artefact

Any document, source code, configuration, checkpoint or other asset that contributes to the engineering of the system.

Within Lumen, documentation is considered an engineering artefact.

---

## Trust through Accuracy

Trust is earned by consistently describing the system as it is today. Documentation must distinguish clearly between implemented functionality, work in progress, architectural direction, and future vision. Aspirational features must never be presented as current capabilities.