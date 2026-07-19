# Lumen

**Lumen is an AI orchestration and continuity engine.**

Modern language models are exceptionally capable, but they remain constrained by finite context windows, isolated chat sessions, limited continuity, and the difficulty of maintaining a reliable understanding across long-running work.

Lumen exists to solve those problems.

Rather than replacing models, clients or tools, Lumen sits between them, preserving the information required for work to continue consistently over hours, days or even months. Its purpose is to make AI-assisted engineering and knowledge work more reliable, auditable and repeatable.

---

## The problem

Today's AI systems generally operate within the boundaries of a single conversation.

As projects grow, important information is gradually lost:

- the original objective
- decisions that have already been made
- reasoning behind those decisions
- assumptions and subsequent corrections
- source provenance
- files already analysed
- unresolved questions
- the model's evolving understanding

Eventually the context window is exhausted, a new conversation begins, and much of that accumulated knowledge must be reconstructed.

For complex engineering work this becomes both inefficient and unreliable.

---

## What makes Lumen different?

Lumen treats the **project**, not the individual model session, as the primary unit of continuity.

Instead of relying solely on whatever remains inside a model's context window, Lumen maintains an external, persistent understanding of the work being performed.

This includes preserving:

- objectives
- established facts
- architectural decisions
- reasoning
- provenance
- checkpoints
- continuity state
- model understanding

By separating continuity from any individual model session, Lumen aims to provide seamless progression across context boundaries while remaining independent of the underlying language model.

---

## Design principles

Lumen is built around a small number of engineering principles.

- **Model independent** — designed to work with different language models rather than being tied to one.
- **Client independent** — compatible with different user interfaces and development environments.
- **Persistent** — preserve continuity beyond the lifetime of a single conversation.
- **Auditable** — important decisions and reasoning should be traceable.
- **Evidence based** — conclusions should be supported by source provenance wherever practical.
- **Practical** — optimise for operational usefulness and trustworthiness rather than benchmark performance.

---

## Repository status

This repository documents the design and evolution of Lumen.

At present it should be regarded as an architectural and engineering project rather than a released software product.

Documentation will evolve alongside implementation.

---

## Learn more

Additional documentation is available in the `docs/` directory.

- **Vision** — why Lumen exists and the philosophy behind the project.
- **Architecture** — the architectural principles and technical design.
- Additional documentation will be added as development progresses.

---

## Licensing

No community licence has yet been published.

The project is currently made available to document its architecture and development.

An **Illuminates Community License** and an **Illuminates Commercial License** are planned and will be published before the first community software release.

---

## Contributions

Community interest, discussion and feedback are welcome.

Formal code contributions are not currently being accepted while the architecture continues to evolve.

A contribution process will be introduced when the project reaches an appropriate level of maturity.

---

## About Illuminates.One

Lumen is developed as part of the Illuminates.One initiative.

Illuminates.One exists to create tools that help people discover, preserve and build upon knowledge. The principles of continuity, provenance and engineering trustworthiness underpin every project developed under the initiative.

Lumen is the first project to embody that philosophy, applying those principles to AI-assisted engineering and knowledge work through orchestration, continuity and persistent project knowledge.