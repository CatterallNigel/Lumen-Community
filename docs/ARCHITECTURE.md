# Architecture

This document describes the conceptual architecture of Lumen.

It defines the major architectural components, their responsibilities and the relationships between them. The architecture is intentionally independent of any particular programming language, language model or deployment environment.

Where appropriate, capabilities are identified as:

- **Current** — implemented.
- **In Development** — actively being developed.
- **Architectural Direction** — planned as part of the long-term architecture.

---

# Architectural Overview

Lumen acts as an orchestration layer between users, clients, language models and external tools.

Its primary responsibility is preserving project continuity while coordinating the interactions required to complete complex tasks.

Unlike a language model, Lumen does not perform inference itself.

Instead, it manages the information and orchestration required for language models to operate consistently across sessions and context boundaries.

```
                    +----------------+
                    |     Client     |
                    +----------------+
                             |
                             v
                  +----------------------+
                  |        Lumen         |
                  | Orchestration Layer  |
                  +----------------------+
                   |    |     |      |
                   |    |     |      |
                   v    v     v      v
              Models  Tools Storage  Policies
```

---

# Architectural Components

## Clients

**Purpose**

Provide the user interface through which people interact with language models.

Examples include desktop applications, IDE integrations, command-line interfaces and future web-based interfaces.

Lumen is designed to remain independent of any specific client.

****Status****

Current.

---

## Lumen Orchestration Layer

**Purpose**

Coordinates communication between clients, language models, tools and persistent project knowledge.

Responsibilities include:

- session management
- continuity management
- context monitoring
- checkpoint creation
- prompt orchestration
- provenance recording
- policy enforcement

The orchestration layer is the core of the Lumen architecture.

****Status****

Current.

---

## Model Providers

Language models perform inference.

Lumen intentionally separates orchestration from inference, allowing different models to be used without changing the surrounding architecture.

Model independence is a core architectural principle.

****Status****

Current.

---

## Tools

Language models frequently require external capabilities such as file access, search, databases or APIs.

Lumen coordinates these tools while maintaining continuity and provenance.

Tool integration remains independent of any individual language model.

****Status****

Current.

---

## Projects

A project represents the persistent unit of work.

Projects survive across multiple sessions and preserve the continuity required for long-running engineering and knowledge work.

Projects are independent of individual model sessions.

****Status****

Current.

---

## Sessions

A session represents a single interaction with an underlying language model.

Sessions are intentionally transient.

Multiple sessions may contribute to the same project throughout its lifetime.

****Status****

Current.

---

# Continuity Management

## Context Monitoring

Lumen monitors available context throughout the lifetime of a session.

Rather than reacting after context has been exhausted, Lumen attempts to preserve continuity before important information is lost.

****Status****

Current.

---

## Checkpoints

Checkpoints preserve a snapshot of a project's continuity state.

They provide durable recovery points that allow work to continue consistently after interruption or across session boundaries.

****Status****

Current.

---

## Distillations

Distillations reduce context consumption while preserving the information required to continue work.

They complement checkpoints by providing efficient continuity rather than replacing original project knowledge.

****Status****

In Development.

---

## Persistent Storage

Continuity artefacts are persisted independently of individual language model sessions.

Persistent storage enables continuity across application restarts and extended periods of work.

Storage technology is considered an implementation detail rather than an architectural dependency.

****Status****

In Development.

---

## Session Rollover

When appropriate, Lumen will transition work from one language model session to another while preserving project continuity.

The objective is for session boundaries to become operational rather than user-visible events.

****Status****

Architectural Direction.

---

# Provenance

Provenance records the origin of information used throughout a project.

Preserving provenance enables decisions to be traced back to their supporting evidence and assists with auditing, validation and future review.

****Status****

In Development.

---

# Prompt and Policy Management

Prompt templates and orchestration policies are treated as versioned engineering artefacts.

Their effectiveness may be evaluated over time using evidence gathered from completed projects.

This allows orchestration behaviour to evolve through measured improvement rather than ad hoc modification.

****Status****

Architectural Direction.

---

# Auditor Models

Future versions of Lumen may employ independent language models to evaluate summaries, reasoning, source coverage or continuity quality.

These auditor models support validation rather than replacing the primary model.

****Status****

Architectural Direction.

---

# Multi-user Architecture

Lumen is currently designed around individual projects.

Future architectural development will support multiple users sharing common infrastructure while maintaining strict isolation of:

- projects
- sessions
- continuity state
- provenance
- permissions
- stored artefacts

The architectural objective is to support collaborative deployment without compromising project independence.

****Status****

Architectural Direction.

---

# Architectural Principles

The architecture is guided by the following principles.

- Project continuity is preserved independently of model sessions.
- Orchestration remains independent of model inference.
- Models, clients and tools remain replaceable.
- Continuity artefacts are durable.
- Provenance is preserved wherever practical.
- Documentation and architecture evolve together.

---

# Summary

Lumen is an orchestration architecture rather than an inference engine.

Its purpose is to preserve project continuity while coordinating the interaction between people, language models, tools and persistent project knowledge.

The architecture is intentionally designed to remain applicable as language models, clients and deployment environments continue to evolve.