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

One concept should have one preferred term, and one term should describe only one concept.

---

## Eliminate ambiguity

Architectural terminology must be precise and unambiguous.

Every defined term should have one meaning throughout the documentation. Likewise, each architectural concept should have one preferred term.

If a term could reasonably be interpreted in multiple ways, it should either be refined or explicitly defined in the project dictionary.

Reducing ambiguity improves implementation consistency, architectural discussion and long-term maintainability.

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

## Knowledge Branch

A Knowledge Branch is a line of understanding that originates from a specific checkpoint within a project.

It inherits all knowledge captured by its originating checkpoint and allows new work to begin without repeating the effort required to establish that understanding.

Multiple Knowledge Branches may originate from the same checkpoint while preserving complete provenance and immutable history.

---

## Lumen Project

The software project that develops Lumen itself.

Unless explicitly stated otherwise, the term *project* throughout the Lumen documentation refers to a project being managed by Lumen rather than the development of Lumen itself.

---

## Session

A continuous execution using a single model context.

A session begins when a new model context is created and ends when that context is no longer continued.

Each session is constrained by a single model context, although Lumen may extend its useful lifetime through checkpointing and context optimisation.

Projects persist beyond sessions.

---

## Objective

The purpose or goal that the project is attempting to achieve.

Objectives should remain stable throughout the lifetime of the project unless intentionally revised.

---

## Checkpoint

A recorded, immutable representation of the project's current understanding and continuity state at a particular point in time.

Checkpoints enable continuity restoration and may become the origin of one or more Knowledge Branches.

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

---

## Document Review Gate

Every new document, and every substantial revision to an existing document, should be reviewed against the following questions:

1. **Does it still reflect reality?**
   The document must clearly distinguish between what is currently implemented, what is in development, and what remains architectural direction or research.

2. **Is it duplicated elsewhere?**
   Information should have one authoritative home. Other documents should reference that source rather than repeating and gradually diverging from it.

3. **Does it introduce concepts that belong in another document?**
   New ideas should be placed according to their purpose: vision, architecture, roadmap, research, design, engineering history, or operational guidance.

4. **Does it reference the documents it depends on?**
   Related concepts should be linked so the documentation forms a connected body of knowledge rather than a collection of isolated files.

5. **Is every architectural term unambiguous?**
   Every defined term should have exactly one meaning. Where ambiguity exists, refine the terminology or extend the dictionary.

6. **Should it become part of the White Paper?**
   Material that explains Lumen’s central problem, principles, architecture, evidence, or significance should be considered for inclusion in the consolidated White Paper.

A document should not be treated as complete merely because its content is correct in isolation. It must also be correctly positioned, connected, current, and consistent with the wider documentation set.

> Lumen is intended to reduce model drift. Its documentation must be designed to resist project drift.

---

## Engineering Integrity

The Engineering Diary is an evidence-based research record.

It is intended to document what was attempted, what was observed and what was learned, regardless of whether the outcome was successful.

Entries must accurately record:

- Objectives
- Hypotheses
- Implementation
- Observations
- Results
- Conclusions
- Outstanding Questions

Negative results are considered equally valuable to positive results.

A disproven hypothesis eliminates future uncertainty and contributes to understanding.

The purpose of the diary is not to demonstrate success, but to preserve engineering knowledge.
