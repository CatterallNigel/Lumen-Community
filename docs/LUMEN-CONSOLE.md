# Architecture

> **Note:** This document is a revised architecture draft generated from the existing architecture and the design decisions made throughout the development of Lumen. It describes the conceptual architecture of the system rather than implementation details and is intended to evolve alongside the project.

# Architecture

This document describes the conceptual architecture of Lumen.

It defines the major architectural components, their responsibilities and the relationships between them. The architecture is intentionally independent of any programming language, language model or deployment environment.

Capabilities are identified as:

- **Current** — implemented.
- **In Development** — actively being developed.
- **Architectural Direction** — planned.

---

# Architectural Principles

Lumen is built around several long-term architectural principles.

These principles guide design decisions regardless of implementation details.

## Continuity

Lumen exists to preserve continuity across context boundaries.

Continuity includes objectives, architectural understanding, cognitive state, reasoning, assumptions, provenance, checkpoints and session history.

---

## Observability

Observability is considered a first-class architectural concern.

Lumen should make its operational and cognitive state visible while work is being performed.

Operational visibility is intended to support engineering confidence, diagnostics and future administrative capabilities.

---

## Separation of Concerns

Major architectural responsibilities remain independent wherever practical.

Examples include:

- Context management
- Checkpoint generation
- Persistence
- Provider interaction
- Session management
- Console presentation

Each subsystem should evolve independently through well-defined interfaces.

---

## Capability Abstraction

Lumen presents capabilities rather than implementation details wherever practical.

For example:

```
Session Persistence : Ready
```

rather than

```
MongoDB : Connected
```

Service health explains capability health but does not define it.

This abstraction allows underlying implementations to evolve without altering the operational model presented to the operator.

---

# Principles for Prompt Design During Long-Running Tasks

The quality of a model's observable reasoning depends not only on the wording of individual prompts but also on how those prompts relate to the model's current cognitive state.

The following principles have emerged through experimentation during the development of Lumen.

---

## Minimise ambiguity

Prompts should communicate a clear objective while allowing the model sufficient freedom to determine the most appropriate response.

Overly prescriptive prompts can unnecessarily constrain reasoning, while ambiguous prompts may produce inconsistent behaviour.

The objective should be explicit, but the path to achieving it should remain flexible unless a specific methodology is required.

---

## Avoid unintended contradictions

Every new instruction should be considered in the context of all preceding System and Assistant prompts.

Even when individual instructions appear reasonable in isolation, their combined effect may alter the model's overall reasoning behaviour.

Prompt changes should therefore be evaluated as changes to an interacting system rather than as isolated edits.

---

## Respect the model's current cognitive state

Prompts should ask only questions that can be answered from the information currently available to the model.

A prompt should not require knowledge that has not yet been observed.

Incremental reasoning should therefore be grounded in present understanding rather than future prediction.

---

## Avoid asking the model to predict future understanding

An incremental checkpoint should preserve the model's current understanding.

It should not require the model to speculate about conclusions that may emerge only after additional source material has been analysed.

Questions concerning future architectural understanding, unresolved questions, or likely refactoring opportunities may be appropriate after task completion, but are generally unsuitable during incremental analysis.

---

## Recognise the difference between human and model information acquisition

Human engineers rarely analyse software by reading every source file sequentially.

Instead they navigate selectively, moving between functions, types, call sites, documentation and related files until they develop sufficient understanding.

By contrast, an LLM performing incremental source analysis acquires knowledge in the order information is presented.

Its understanding therefore develops cumulatively rather than through selective exploration.

Prompt design should acknowledge this difference rather than assuming human reading strategies.

---

## Ask questions appropriate to the current stage of analysis

The usefulness of a question depends upon when it is asked.

For example:

- "What does the model currently understand?" is appropriate throughout incremental analysis.

- "What architectural refactoring is required?" may only become meaningful after the relevant architecture has been observed.

Prompt timing is therefore as important as prompt wording.

---

## Evaluate behavioural impact rather than prompt wording

Small prompt changes can produce disproportionately large changes in observable reasoning behaviour.

Prompt optimisation should therefore be evaluated experimentally rather than intuitively.

The objective is not simply to produce better wording, but to produce better observable reasoning.

---

## Respect how the model acquired its knowledge

Prompt design should respect not only what the model knows, but how it came to know it.

Large Language Models performing incremental analysis acquire knowledge sequentially as information is presented.

Unlike a human engineer, they cannot arbitrarily navigate to unseen portions of the source material to answer a question.

Consequently, prompt design should not assume the model possesses information that has not yet been encountered.

Questions requiring knowledge beyond the model's current observable state should generally be deferred until sufficient source material has been analysed.

The sequence in which knowledge is acquired is therefore an important consideration when designing prompts for long-running tasks and Cognitive Checkpoints.

---

# Summary

These principles have emerged through observation and experimentation rather than theoretical prompt design.

They should therefore be regarded as engineering guidance rather than immutable rules and should continue to evolve as further behavioural experiments are performed.

The overarching objective remains unchanged:

> **Prompt design should maximise the quality of a model's observable reasoning while remaining consistent with the model's current state of knowledge.**

Lumen therefore treats prompt engineering as an evidence-based engineering discipline, where prompt behaviour is measured, compared and refined through repeatable experimentation rather than intuition.

---

# Observability

Lumen treats observability as a fundamental architectural capability rather than a debugging facility.

The purpose of observability is to allow an operator to understand the current operational state of a running Lumen instance without requiring inspection of implementation details or diagnostic logs.

Observability therefore includes visibility into:

- Operational state
- Cognitive state
- Session progression
- Checkpoint progression
- Provider interactions
- Service capability assessment
- Operational warnings
- Runtime activity

The primary objective is to answer one question:

> **What is Lumen doing now?**

Observability supports engineering confidence, diagnostics and future operational capabilities.

Administrative control remains a separate architectural concern.

---

# Operational State

Lumen distinguishes between operational state and operational artefacts.

## Operational State

Operational state represents the current activity of a running Lumen instance.

Examples include:

- Current session
- Current objective
- Current task
- Current checkpoint
- Current provider
- Current operational mode
- Current capabilities
- Current service interactions

Operational state is transient and continually changes as work progresses.

---

## Operational Artefacts

Operational artefacts are permanent records created while work was performed.

Examples include:

- Historical sessions
- Historical checkpoints
- Results
- Distillations
- Logs
- Persistence history

Operational artefacts are preserved for traceability, continuity and engineering analysis.

---

# Session-Centric Observability

Observability within Lumen is session-centric.

Only the active session represents the current operational state.

Historical sessions remain permanently available but are presented separately from the active session.

The Lumen Console therefore distinguishes between:

- Active Session
- Session History

This separation prevents historical work from appearing to be current activity while preserving complete historical traceability.

The guiding principle is:

> **The Console answers "What is happening now?" before "What happened previously?"**

---

# Lumen Console

The Lumen Console is the primary operational interface for a running Lumen instance.

The console exists to provide real-time visibility into Lumen's operational, cognitive and orchestration state.

The console is intentionally read-only by default.

Its primary responsibility is operational observability rather than operational control.

The Console should allow an operator to understand:

- What Lumen is currently doing.
- Which capabilities are currently available.
- Which services are currently interacting.
- Which provider is active.
- The progress of the current task.
- The current checkpoint state.
- Session progression.
- Operational warnings.
- Current operational mode.

Historical sessions and checkpoints remain accessible but are presented separately from the active operational view.

The architecture, operational philosophy and user interface design of the console are documented separately in:

```
docs/LUMEN-CONSOLE.md
```

---

# Architectural Direction

Lumen continues to evolve around three complementary architectural pillars.

## Continuity

Preserving knowledge across context boundaries.

Examples include:

- Sessions
- Checkpoints
- Distillations
- Persistence
- Recall

---

## Observability

Making Lumen's behaviour visible while work is being performed.

Examples include:

- Lumen Console
- Operational capability assessment
- Service health
- Runtime activity
- Checkpoint progression
- Provider interactions

---

## Control

Administrative capabilities built upon the observability layer.

Examples include:

- Provider management
- Session management
- Runtime configuration
- Operational administration

Control capabilities are considered a future architectural direction and are intentionally separated from observability.