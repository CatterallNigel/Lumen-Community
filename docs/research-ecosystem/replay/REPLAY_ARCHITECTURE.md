# Lumen Replay (Repetere)
## Runtime Architecture and Product Responsibilities

Version: 1.0
Status: Architectural Baseline

---

# Overview

Lumen Replay is **not** an HTTP replay engine.

Replay exists to reproduce a previously recorded AI conversation under identical starting conditions in order to determine whether the model follows the same behavioural path.

Replay is deliberately **not** responsible for assessing model quality or determining whether one reasoning path is better than another.

Those responsibilities belong to Lumen Assess.

---

# Product Responsibilities

The Lumen engineering toolchain consists of four independent products.

```
Trace (Vestigare)
        ↓
Replay (Repetere)
        ↓
Assess (Aestimare)
        ↓
Servire
```

Each product has a single, clearly defined responsibility.

---

# Trace

Trace captures everything that transpires between Pi and Lumen.

Responsibilities:

- record every HTTP request
- record every HTTP response
- record tool calls
- record tool results
- record heartbeats
- record checkpoint polling
- record operational traffic
- never modify recorded data

Trace remains entirely passive.

It records reality.

---

# Replay

Replay reconstructs a replayable conversation from the Trace recording.

Replay removes transport noise that has no influence on model behaviour.

Examples include:

- heartbeat traffic
- checkpoint polling
- operation polling
- health requests
- transport keep-alives

Replay creates a Replay Plan representing the meaningful conversational events.

Replay then executes the conversation against Lumen under controlled conditions.

Replay compares every meaningful event with the original recording.

Replay **does not** determine whether one behaviour is better than another.

---

# Assess

Assess evaluates Replay outcomes.

Assess combines:

- original Trace recording
- Replay execution metadata
- Lumen session data
- checkpoints
- summaries
- tool usage
- final answers

Assess determines:

- answer quality
- behavioural differences
- tool efficiency
- checkpoint evolution
- consistency
- model capability
- model stability

---

# Servire

Servire orchestrates the complete engineering workflow.

---

# Replay Runtime

Replay operates in two distinct phases.

## Phase 1

Replay owns the conversation.

```
Replay
    ↓
Lumen
    ↓
Model
```

Replay compares every meaningful conversational event with the prepared Replay Plan.

Transport traffic is ignored.

---

## Matching Conversation

If every event matches the recorded conversation:

```
Ask

↓

Tool Call

↓

Tool Result

↓

Answer
```

Replay records:

```
MATCH
```

The replay completes.

No data is forwarded to Trace.

No tooling is required from Pi.

---

## Fork Point

Replay continuously compares the live conversation against the recorded Replay Plan.

The first meaningful behavioural difference is known as the **Fork Point**.

Examples include:

- different tool selected
- different tool arguments
- additional tool calls
- omitted tool calls
- different assistant response

Replay records:

- last matching step
- first divergent step
- expected event
- observed event

Replay has now completed its experimental objective.

---

## Phase 2

Following the Fork Point Replay becomes transparent.

```
Pi
    ↓
Trace
    ↓
Replay
    ↓
Lumen
```

Replay no longer attempts comparison.

Replay simply forwards traffic.

Trace begins recording.

Pi provides tools.

Lumen continues the conversation normally until completion.

Replay records execution metadata only.

---

# Why Replay Stops Comparing

Replay exists to answer one question.

> How far does the model reproduce the original behavioural path before choosing a different one?

Everything after the Fork Point represents a new behavioural path.

Continuing comparison no longer provides meaningful information.

Instead Replay allows the new conversation to complete naturally.

Assess later evaluates the significance of that new path.

---

# Replay Outputs

Replay produces:

- Replay Plan
- Replay Result
- Fork Point
- Matching Steps
- Divergent Step
- Replay Metadata

Replay deliberately does **not** produce behavioural judgements.

---

# Architectural Principle

Trace captures what happened.

Replay reproduces what mattered.

Assess determines what it means.

These responsibilities should remain independent throughout future development.