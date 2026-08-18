# Lumen Replay Runtime Architecture

Version: 1.1  
Status: Architectural baseline (Milestone 10)  
Applies to: Lumen Replay (Repetere)

---

# Product boundary

> **Trace captures what happened. Replay reproduces what mattered. Assess determines what it means.**

Trace remains the immutable recorder of all traffic between Pi and Lumen.

Replay derives a behavioural Replay Plan from a Trace recording, reproduces the original opportunity for the model to solve the task, detects the first behavioural divergence (Fork Point), and then transparently allows the live conversation to continue.

Assess later combines Trace, Replay and Lumen evidence to evaluate behaviour, quality, stability, efficiency and capability.

Replay deliberately **does not**:

- score answers;
- judge reasoning quality;
- determine whether a divergent path is better;
- assess intelligence.

Replay answers one question only:

> **Did the model behave the same way?**

---

# Behavioural Replay

Replay reproduces **observable model behaviour**, not conversational presentation.

Observable behaviour consists of:

- tool selection
- tool arguments
- tool ordering
- tool results
- final observable outcome

Replay intentionally ignores presentation artefacts including:

- assistant prose
- reasoning summaries
- thinking output
- streamed formatting
- heartbeat messages
- progress updates
- Lumen checkpoints
- timing differences
- generated tool-call identifiers

These remain useful diagnostic information and may be logged, but they never cause a Replay Fork.

---

# Runtime topology

Normal deployment requires a single routing change in Trace.

```text
Pi
    ↓
Trace
    ↓
Replay
    ↓
Lumen
    ↓
Model Provider
```

Replay is a transparent proxy by default.

Requests and responses are forwarded unchanged until an explicit:

```text
\obt replay start <replay-id>
```

command is received.

---

# Runtime states

```text
                 TRANSPARENT
                      │
                      │
      \obt replay start <replay-id>
                      │
                      ▼
                 COMPARING
                  │      │
      complete    │      │ behavioural fork
      replay      │      │
                  ▼      ▼
             TRANSPARENT PASSTHROUGH
                             │
                             │ final response /
                             │ stop /
                             │ cancel
                             ▼
                        TRANSPARENT
```

---

# Transparent

Replay behaves as a transparent proxy.

All traffic passes directly between Trace and Lumen.

Replay performs no behavioural comparison.

---

# Comparing

Replay loads a prepared Replay Plan derived from a Trace recording.

Replay then privately reproduces the recorded model interaction.

For every model response Replay observes:

- tool calls
- tool arguments
- tool ordering

Replay ignores conversational presentation.

Generated tool-call identifiers are ignored.

Tool arguments are canonicalised before comparison to eliminate formatting differences that do not change behaviour.

---

# Replay Plan

A Replay Plan consists only of observable behavioural events.

For Pi these events are:

```text
Recorded Request

↓

Model Tool Call

↓

Recorded Tool Result

↓

Recorded Request

↓

Model Tool Call

↓

...

↓

Final Model Response
```

Replay therefore compares behaviour rather than messages.

---

# Nominal replay

During successful replay Pi is not required.

Replay submits the recorded request to Lumen.

When the model produces the expected tool call:

```text
Model Tool Call
        │
        ▼
Replay validates behaviour
        │
        ▼
Replay returns recorded tool result
        │
        ▼
Replay advances to next Replay Plan step
```

The next recorded request already contains the cumulative conversation together with the original tool result.

Replay therefore reproduces the original execution path without requiring Pi until behavioural divergence occurs.

---

# Behaviour matching

Replay considers a behavioural step to match when:

- tool name matches;
- canonical tool arguments match;
- tool ordering matches.

Replay intentionally ignores:

- assistant wording;
- reasoning text;
- streamed presentation;
- progress output;
- heartbeat messages;
- checkpoint messages;
- formatting;
- timing.

These are logged only for diagnostics.

---

# Fork Point

The first observable behavioural difference is the **Fork Point**.

Replay records:

- run identifier;
- source recording identifier;
- prepared Replay identifier;
- number of matching behavioural steps;
- first divergent behavioural step;
- expected tool summary;
- observed tool summary;
- detection timestamp.

Replay immediately returns the observed tool call unchanged through Trace to Pi.

Replay then becomes a transparent proxy.

Pi performs tool execution.

Lumen continues the live conversation.

Trace records everything from the Fork Point onwards.

Replay performs no further intervention.

---

# Match completion

When every recorded behavioural step matches:

- Replay records a successful run;
- Replay records the number of matching behavioural steps;
- Replay returns a concise completion message to Pi.

The recorded nominal conversation is **not** replayed through Trace because it already exists within the original Trace recording.

---

# Final response

Replay observes the model until one of two terminal conditions occurs.

## Complete behavioural match

All recorded behavioural steps match.

Replay completes successfully.

## Behavioural fork

A different observable tool behaviour occurs.

Replay records the Fork Point.

Replay becomes transparent.

The live interaction continues normally until the model produces its final response.

Replay then completes.

---

# Explicit commands

Replay consumes these commands locally.

They are **never** forwarded to Lumen.

```text
\obt replay

\obt replay start <replay-id>

\obt replay status

\obt replay stop
```

Prepared Replay sessions continue to be created through the Replay engineering UI or REST API.

---

# Logging

Replay maintains two independent logging streams.

## Operational Log

Records:

- replay lifecycle;
- replay progress;
- replay completion;
- replay cancellation;
- fork detection;
- transition into transparent mode.

## Command Decision Log

Records:

- operator command;
- command classification;
- routing decision;
- forwarding decision;
- Replay handling decision;
- routing rationale.

These logs are intended to support engineering, debugging and behavioural analysis.

---

# Persistence

Replay-owned runtime evidence is stored within:

```text
replay_runs
```

Replay never modifies Trace recordings.

Trace collections remain read-only.

Replay records only:

- Replay runs;
- behavioural matches;
- Fork Point evidence;
- Replay metadata.

---

# Design principles

Replay is founded on the following engineering principles.

### Replay compares behaviour, not presentation.

Presentation may change without behavioural significance.

Behaviour consists of observable actions.

---

### Replay determines *whether* behaviour diverged.

Replay does not determine whether the divergence is good or bad.

That responsibility belongs to Assess.

---

### Replay never alters the live conversation after a Fork Point.

Once behavioural divergence occurs:

- Replay records the event;
- Replay becomes transparent;
- Trace captures the remainder of the interaction.

---

### Replay is deterministic.

Given the same Trace recording and the same observable model behaviour, Replay will always reach the same behavioural decision.

---

# Relationship to Assess

Replay and Assess have deliberately separate responsibilities.

Replay answers:

> **Did the behaviour change?**

Assess answers:

> **Was the new behaviour better, worse, equivalent or simply different?**

Keeping these responsibilities separate ensures Replay remains a simple, deterministic behavioural experiment controller while Assess becomes the behavioural analysis engine.

---

# Known boundary

Replay currently buffers upstream responses so complete OpenAI-compatible JSON or SSE evidence can be examined before deciding whether to suppress a nominal response or expose a Fork Point.

A future runtime-hardening milestone may introduce bounded disk-backed buffering for extremely large streamed responses.

---

# Summary

Replay is a behavioural experiment controller.

It reproduces recorded model behaviour under controlled conditions, detects the first observable behavioural divergence, records objective evidence of that divergence, and then transparently allows the live conversation to continue.

Replay does not evaluate quality.

Replay does not evaluate intelligence.

Replay simply determines whether the model behaved the same way.