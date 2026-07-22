# Architecture

> **Note:** This document is a revised architecture draft generated from the existing
architecture and the design decisions made during the v3.2.4 checkpoint experiments.
It is intended to replace the current ARCHITECTURE.md after review.

# Architecture

This document describes the conceptual architecture of Lumen.

It defines the major architectural components, their responsibilities and the relationships
between them. The architecture is intentionally independent of any programming language,
language model or deployment environment.

Capabilities are identified as:

- **Current** — implemented.
- **In Development** — actively being developed.
- **Architectural Direction** — planned.

---

# Architectural Overview

Lumen is an orchestration layer that preserves project continuity while coordinating
language models, tools and persistent knowledge.

Unlike a language model, Lumen does not perform inference.

Instead it manages:

- continuity
- provenance
- checkpointing
- project state
- orchestration
- prompt policy
- persistent knowledge

The language model owns inference.

Lumen owns continuity.

---

# Core Execution Model

Project
    └── Knowledge Branch
            └── Session
                    └── Context

## Project

A Project is the durable unit of engineering work.

Projects persist independently of any model session and own:

- knowledge branches
- checkpoints
- distillations
- provenance
- engineering metadata

Status: Current

---

## Knowledge Branch

A Knowledge Branch represents a continuous lineage of understanding.

Every branch begins from a checkpoint.

Multiple branches may originate from the same checkpoint without modifying history.

Status: Current

---

## Session

A Session represents one continuous execution using a single model context.

Sessions are transient.

Projects and Knowledge Branches are persistent.

Status: Current

---

## Context

Context is the finite working memory supplied by the language model.

The model defines the context window.

Lumen optimises what occupies it.

Status: Current

---

# Continuity

Lumen preserves continuity through immutable checkpoints and distilled understanding.

The objective is preserving understanding rather than preserving conversation.

---

# Checkpoints

A checkpoint is an immutable snapshot of the model's observable cognitive state.

It captures the model's current understanding independently of any future task.

Checkpoints support:

- recovery
- continuity
- engineering analysis
- knowledge branching
- benchmarking

Status: Current

---

## Final Cognitive Checkpoint

After all requested source material has been processed, Lumen captures one final checkpoint
before the model begins generating the requested response.

This checkpoint records the model's completed understanding before task-specific reasoning.

Benefits:

- complete observable cognitive state
- canonical recovery point
- reproducible engineering measurement
- clean separation between understanding and execution

Status: In Development

---

## Checkpoint Evolution

Successive checkpoints may be analysed to understand how knowledge evolves.

Analysis includes:

- stable understanding
- refined understanding
- newly acquired understanding
- contradictions
- confidence changes
- continuity quality

Checkpoint evolution forms an engineering measurement rather than a textual diff.

Status: In Development

---

## Distillations

Distillations reduce context consumption while preserving continuity.

Status: In Development

---

# Prompt Management

Prompt templates are versioned engineering artefacts.

Checkpoint prompts are treated as engineering components and are expected to evolve through
controlled experimentation.

Evaluation includes:

- architectural understanding
- uncertainty identification
- checkpoint quality
- continuity quality
- final task quality

Prompt revisions are benchmarked against previous versions.

Status: Architectural Direction

---

# Auditor Models

Independent reviewer models may evaluate checkpoint quality, continuity quality and source
coverage.

Reviewer output remains separate from checkpoints.

Reviewer models never modify:

- checkpoints
- continuity
- project history

Human engineers remain the final decision makers.

Status: Architectural Direction (planned for v3.2.7 implementation)

---

# Architectural Principles

1. The model defines the context window.
2. Lumen defines continuity.
3. Projects are the durable unit of work.
4. Knowledge Branches represent evolving understanding.
5. Sessions are execution containers.
6. Checkpoints are immutable.
7. Every checkpoint may create a new Knowledge Branch.
8. Understanding is separated from execution.
9. Final Cognitive Checkpoints capture complete understanding before task execution.
10. Prompt evolution is evidence-driven.

---

# Summary

Lumen is an orchestration architecture rather than an inference engine.

Its purpose is to preserve, measure and evolve project understanding across finite model
contexts while remaining independent of models, clients and deployment technologies.
