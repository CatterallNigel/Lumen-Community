# LUMEN-CONSOLE-v2

**Status:** Historical
**Current reference:** ../../architecture/core/ARCHITECTURE.md


## The Lumen Operational Console

### Purpose

The Lumen Operational Console is the primary operational interface for the Lumen execution engine. Its purpose is to provide immediate visibility into the live state of an executing system rather than requiring an engineer to reconstruct behaviour from log files after execution has completed.

The console answers three questions:

1. What is Lumen doing?
2. Why is it doing it?
3. What is it likely to do next?

## Architectural Principles

The console is built around three principles:

- **Execution before presentation** – the console reflects the execution engine rather than driving it.
- **Operational awareness** – important information should be visible without inspecting logs.
- **Capability abstraction** – present capabilities rather than implementation details.

## The Three Pillars

### Continuity
Preserves knowledge across bounded context windows through sessions, checkpoints, distillation and recall.

### Execution
Coordinates models, providers, tools, objectives, recovery and orchestration.

### Observability
Makes execution understandable while it is happening.

## Operational Model

```
Project
└── Session
    └── Run
```

A Project represents a long-lived body of work, a Session is a bounded conversation or analysis, and a Run is a single execution of an objective.

## Live Operational State

The console should present:

- Active project
- Active session
- Active run
- Current capability
- Current objective
- Current execution phase
- Runtime
- Active model and provider

## Source Acquisition

When analysing source code, the console should expose:

- Current file
- Read progress
- Current offset
- Outstanding read request
- EOF verification state

This allows engineers to distinguish between active analysis and stalled execution.

## Continuity

The console should indicate:

- Current Cognitive Checkpoint
- Final Cognitive Checkpoint status
- Distillation status
- Session continuity state

## Recovery

Operational recovery should be visible, including:

- Replay Guard status
- Recovery state
- Recovery attempts
- Current recovery strategy

## Persistence

Persistence is treated as a pipeline rather than a single event.

Visible states include:

- Pending
- Persisting
- Persisted
- Retry pending
- Failed

## Operational Timeline

A chronological timeline provides a concise history of the current run.

Example:

```
Reading source
EOF verified
Building architectural understanding
Generating response
Creating Final Cognitive Checkpoint
Persisting
Complete
```

## Future Direction

The Operational Console is intended to become the primary engineering interface for Lumen. It complements historical checkpoint viewers by exposing live execution state, enabling engineers to understand the orchestration engine as it operates rather than after it has finished.
