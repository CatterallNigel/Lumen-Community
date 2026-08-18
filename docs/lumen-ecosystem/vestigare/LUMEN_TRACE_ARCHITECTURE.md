# LUMEN_TRACE_ARCHITECTURE.md

# Lumen Trace Architecture

## Overview

Lumen Trace is the recording component of the Lumen engineering toolset.

Its purpose is to transparently record every interaction between Pi and Lumen without altering behaviour. Trace captures requests, responses and protocol metadata, providing a complete record of an AI interaction for later replay and analysis.

Trace is intentionally passive. It observes but never modifies execution.

---

## Responsibilities

Lumen Trace is responsible for:

- Acting as a transparent HTTP proxy.
- Recording every request and response.
- Preserving protocol metadata.
- Persisting recordings to MongoDB.
- Providing a lightweight inspection UI.

Trace does **not** attempt to analyse, replay or score model behaviour.

---

## Architecture

```
Pi
    │
    ▼
Lumen Trace
    │
    ▼
Lumen
    │
    ▼
Qwen / Ollama
```

Every request passing through Trace is recorded before being persisted as part of a recording session.

---

## Recording Model

A recording consists of:

- Recording metadata
- Request messages
- Response messages
- Exchange identifiers
- Timing information

Each request/response pair shares a common Exchange ID, allowing Replay to reconstruct the original conversation.

---

## User Interface

The Trace UI provides a live operational view of the current recording.

It allows an engineer to:

- Start and stop recordings.
- Observe recorded traffic.
- Inspect request and response bodies.
- Review protocol metadata.

The UI is intentionally lightweight and focuses on evidence capture rather than analysis.

---

## Relationship to Other Lumen Components

Trace forms the first stage of the Lumen engineering toolchain.

```
Trace
    │
    ▼
Replay
    │
    ▼
Assess
    │
    ▼
Servire
```

- **Trace** records interactions.
- **Replay** reproduces recorded executions.
- **Assess** evaluates and compares behaviour.
- **Servire** provides operational visibility across the complete ecosystem.

---

## Current Status

Milestones 1–4 are complete.

Implemented features include:

- FastAPI application
- Configuration management
- Structured logging
- MongoDB persistence
- Transparent proxy
- Recording engine
- Live recording UI

Lumen Trace is considered feature complete for its first production-ready release.