# Lumen Research Ecosystem

## Why are we building this?

One of the central research questions behind Lumen is not simply whether an AI model can produce a correct answer, but how that answer develops over time and why different executions of the same model can produce materially different outcomes.

During development we have repeatedly observed that, with the same model, the same prompt, the same files and the same execution environment, answer quality can vary significantly between runs. This raises an important question:

> How does a model decide that it has done enough to answer?

To investigate this scientifically, we need to capture real interactions, replay them under controlled conditions, and objectively compare the results. Rather than embedding experimental logic into Lumen itself, the research platform is divided into a set of small, focused projects, each with a single responsibility.

---

# Naming Convention

The Lumen Research Ecosystem uses two complementary naming conventions.

## Development Names

The development names are used throughout the source code, Git repositories, documentation and internal engineering discussions.

These names are intentionally descriptive and immediately understandable by developers.

Examples include:

- `lumen-trace`
- `lumen-replay`
- `lumen-assess`
- `lumen-servire`

These names prioritise clarity over branding.

---

## Product Names

The public product names use Latin.

| Development Name | Product Name | Meaning |
|------------------|--------------|---------|
| Lumen Trace | **Lumen Vestigare** | To trace, investigate, follow tracks |
| Lumen Replay | **Lumen Repetere** | To repeat, revisit, replay |
| Lumen Assess | **Lumen Aestimare** | To evaluate, estimate, assess |
| Lumen Servire | **Lumen Servire** | To serve |

---

## Why Latin?

The choice of Latin is intentional.

Latin provides a neutral and timeless naming convention that is not associated with any modern nation or spoken language.

Unlike English product names, Latin terminology does not naturally favour one language community over another and is already widely recognised within science, medicine, engineering and law as a language of classification and description.

The Latin names also describe the purpose of each component as an action rather than simply identifying it as a piece of software.

The resulting progression reflects the lifecycle of the Lumen Research Ecosystem.

| Research Stage | Product |
|----------------|---------|
| Observe | **Lumen Vestigare** |
| Reproduce | **Lumen Repetere** |
| Evaluate | **Lumen Aestimare** |
| Operate | **Lumen Servire** |

This progression reinforces the philosophy behind the ecosystem.

The projects are not independent tools.

They form a complete research workflow:

Observe reality.

Reproduce reality.

Evaluate the results.

Operate the ecosystem.

The English names remain the preferred names for software development and repositories, while the Latin names provide a consistent product identity for documentation, presentations and future public releases.

---

# Lumen Trace

Lumen Trace is a transparent recording proxy that sits between Pi and Lumen.

Its responsibility is to capture every interaction exactly as it occurred without modifying the requests or responses.

Lumen Trace records:

- request and response JSON
- session identifiers
- timestamps
- HTTP metadata
- message ordering

Each capture is stored as a named recording and forms an immutable source for future experiments.

Lumen Trace performs **no analysis** and **no replay**.

Its sole purpose is accurate observation.

---

# Lumen Replay

Lumen Replay consumes recordings produced by Lumen Trace.

It reproduces identical interactions with Lumen under tightly controlled conditions.

Replay is responsible for:

- selecting a recorded interaction
- creating new replay sessions
- executing repeated runs
- resetting the model between runs
- recording experiment metadata

Replay intentionally performs no assessment of answer quality.

Its purpose is to generate repeatable experimental data.

---

# Lumen Assessment

Lumen Assessment analyses the outputs generated during replay experiments.

Initially the assessment is intentionally simple:

- identical or different answer
- answer hashes
- answer length
- basic statistics

Over time Assessment will evolve to compare:

- semantic similarity
- reasoning development
- checkpoint evolution
- evidence coverage
- answer quality
- understanding development

Assessment provides objective measurements while remaining independent of Replay.

---

# Lumen Servire

Lumen Servire is the operational dashboard for the Lumen research ecosystem.

It provides a unified interface across the independent projects while keeping their responsibilities separate.

Servire integrates:

- Lumen Trace
- Lumen Replay
- Lumen Assessment

Future versions may also include operational monitoring, experiment management, model lifecycle management and research reporting.

Servire contains no experimental logic itself.

Its purpose is to provide a single operational view of the complete Lumen ecosystem.

---

# Design Philosophy

The ecosystem follows a strict separation of responsibilities.

| Project | Responsibility |
|----------|----------------|
| Lumen | AI orchestration |
| Lumen Trace | Capture |
| Lumen Replay | Controlled experimentation |
| Lumen Assessment | Analysis |
| Lumen Servire | Unified operational dashboard |

This separation keeps every project small, understandable and independently testable while allowing the overall ecosystem to evolve without introducing unnecessary coupling between components.