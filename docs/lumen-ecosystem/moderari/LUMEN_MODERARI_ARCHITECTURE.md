# Lumen Moderari Architecture

## Overview

Lumen Moderari is the orchestration and continuity boundary between the Lumen communication/observability path and the active model provider.

It has a single primary responsibility:

**Maintain and orchestrate the logical model-facing conversation for a Lumen session.**

Moderari understands conversational/model semantics where required for orchestration and context management, but deliberately does not own client presentation, communication-protocol bridging, provider lifecycle, recording, replay, or assessment.

---

# Architectural Position

```text
                         Servire
                            |
                    Operational control
                            |
                            v
Rogare --> Pontis --> Vestigare --> Repetere --> Moderari --> Model Provider
              |                                  |
              | ACP                              | provider requirement
              v                                  v
              Pi                              Praebere
```

The exact active path can vary according to deployment and enabled services, but Moderari remains the model-facing orchestration boundary.

---

# Architectural Responsibility

Moderari owns:

- logical Lumen session continuity;
- model-facing request orchestration;
- conversation/context management;
- model/tool continuation semantics;
- context compaction/summarisation behaviour;
- long-request heartbeat generation;
- model-processing progress reporting;
- operational state relevant to orchestration.

Moderari does not own:

- Rogare UI state;
- ACP transport;
- Pi lifecycle or tool-provider protocol;
- provider process lifecycle;
- Ollama process management;
- Trace/Vestigare recording;
- Replay/Repetere experiment control;
- assessment;
- Servire stack lifecycle.

---

# Request Plane

The primary Moderari request plane is OpenAI-compatible HTTP.

Conceptually:

```text
Upstream Lumen component
        |
        | OpenAI-compatible HTTP
        v
     Moderari
        |
        | translated/provider-compatible request
        v
  Model Provider
```

Moderari may adapt model-facing protocol details where required by the configured translator/provider behaviour, but the logical Lumen session remains stable across that translation.

---

# Session Identity

## Logical Session

Moderari operates against a logical Lumen session identifier.

For Rogare conversations this is typically:

```text
session-rogare-YYYYMMDD-HHMMSS
```

The logical session is the continuity key for model-facing behaviour.

It should remain stable across:

- ordinary turns;
- tool calls;
- tool-result continuation;
- client UI reattachment;
- provider bootstrap/re-entry;
- context-management operations.

---

## Session Creation

When a valid originating Lumen session identity is supplied, Moderari should use that identity.

A new Moderari conversation-root identity is appropriate only when no usable external logical session identity exists and a genuinely new conversation is being created.

This distinction is important for bridged clients.

Pontis is responsible for propagating the originating Lumen session identity when an ACP/Pi provider path re-enters the Lumen HTTP path.

Moderari should not maintain an ACP-to-Lumen alias table.

---

# Context Ownership

Moderari is the authoritative model-facing context owner for an active Lumen session.

Context can include:

- prior conversational turns;
- tool-call and tool-result continuation state;
- summaries/compacted history;
- continuity information;
- model-facing system/instruction state;
- information needed to resume the logical conversation.

Client-rendered history is presentation state and must not supersede the authoritative Moderari context.

This enables:

```text
Rogare UI destroyed
        |
        v
logical session remains
        |
        v
Rogare UI recreated
        |
        v
reattach to same Lumen session
```

without creating a new model conversation.

---

# Context Management

Moderari monitors and manages the amount of context supplied to the model.

Its context-management layer may:

- preserve history unchanged when within configured bounds;
- estimate context utilisation;
- compact or summarise history when required;
- preserve important continuity information;
- continue using the same logical session after compaction.

Context transformation must not implicitly create a new logical conversation.

---

# Model Provider Boundary

Moderari consumes a configured model endpoint but does not own the provider process.

```text
Moderari
    |
    | model request
    v
Provider endpoint
    |
    v
Model
```

For the current local deployment:

```text
Moderari --> Ollama --> Qwen
```

The provider/model lifecycle belongs to Praebere.

---

# Relationship to Praebere

Praebere manages model-provider availability.

The architectural boundary is:

```text
Moderari                         Praebere
    |                                |
    | needs model X                  | manages provider/model
    |                                |
    +---------- Provider ------------+
```

Moderari may communicate which supplier/model is required.

Praebere owns:

- provider startup;
- provider shutdown;
- model loading;
- model warm-up;
- model unloading;
- readiness/availability inspection.

Moderari owns use of the resulting model service.

Moderari must not become responsible for launching or killing Ollama.

---

# Tool-Call Continuation

Tool-capable conversations cross multiple Lumen responsibilities.

A simplified flow is:

```text
Rogare
   |
   v
Pontis
   |
   v
Moderari
   |
   v
Model
   |
   | tool call
   v
Pontis
   |
   | ACP
   v
Pi
   |
   | tool result / continuation
   v
Pontis
   |
   v
Moderari
   |
   v
Model
```

Vestigare and Repetere can remain in the HTTP path according to the active stack configuration.

Moderari understands enough of the model/tool continuation to carry the conversation forward, but it does not manage the ACP session itself.

---

# Relationship to Pontis

Pontis and Moderari have complementary responsibilities.

## Pontis

Pontis owns:

- client/provider communication bridging;
- HTTP proxying;
- ACP communication;
- ACP session lifecycle;
- Pi integration;
- Lumen-to-ACP session correlation;
- propagation of originating Lumen session identity.

## Moderari

Moderari owns:

- the logical model conversation;
- model-facing context;
- continuation behaviour;
- context management;
- heartbeat/progress generation.

Pontis should not interpret the conversation.

Moderari should not understand the mechanics of ACP.

This follows the Pontis architectural principle that protocol bridging and conversational understanding remain separate concerns.

---

# Backchannel Plane

Moderari produces operational backchannel events independently of the conversational response path.

```text
                   conversational response
Moderari --------------------------------------> upstream
    |
    |
    +---- heartbeat/progress ----> Pontis
                                      |
                                      v
                              session backchannel
                                      |
                                      v
                                  Rogare poll
```

Backchannel information is operational metadata.

It must not alter the conversational response or become model-visible dialogue.

---

# Heartbeat Events

Heartbeat events indicate that a long-running Moderari/model operation remains active.

They are associated with:

- logical Lumen session ID;
- request ID;
- event sequence/timing information as applicable.

The heartbeat does not assert that useful semantic progress has occurred. It establishes liveness.

---

# Progress Events

Progress events provide richer information where available.

They can include information such as:

- model identity;
- elapsed processing time;
- estimated context utilisation;
- current working status.

The elapsed value generated by Moderari describes Moderari/model processing and is not necessarily identical to the client's complete request elapsed time.

This distinction is intentional.

---

# Request Identity

Individual requests have their own request identifiers independently of the logical session.

Conceptually:

```text
Lumen Session
session-rogare-123
    |
    +-- Request A
    +-- Request B
    +-- Tool continuation C
    +-- Request D
```

Request IDs support logging, progress correlation, and diagnostics.

They do not replace the session identity.

---

# Recording and Replay Boundary

Moderari does not own recording or replay.

```text
... --> Vestigare --> Repetere --> Moderari --> Model
```

Vestigare owns recording/provenance capture.

Repetere owns replay matching and fork behaviour.

Moderari should behave as the live orchestrator regardless of whether a request is being observed or replay-controlled upstream.

---

# Operational Control Boundary

Servire is responsible for operational lifecycle management.

Servire may:

- start Moderari;
- stop Moderari;
- monitor Moderari;
- display Moderari status;
- expose Moderari's workspace;
- coordinate stack dependency order.

Moderari does not start or stop the wider Lumen stack.

---

# Failure Boundaries

Moderari should distinguish failures according to responsibility.

Examples include:

## Provider unavailable

The configured model endpoint cannot service the request.

This is a provider availability/lifecycle concern, potentially surfaced through Praebere/Servire.

## Model request failure

The provider accepted the service role but the particular inference request failed.

This belongs to Moderari's model-request handling.

## Tool communication failure

The external tool-provider path fails.

Pontis/Pi owns the communication boundary; Moderari owns how the logical conversation responds to the failed continuation.

## Context-management failure

Moderari cannot safely prepare the required model context.

This is a Moderari orchestration failure.

These boundaries should remain visible in logs rather than being collapsed into a generic model error.

---

# Runtime Responsibilities

Moderari owns:

- HTTP model-orchestration endpoint;
- logical session selection;
- model-facing context;
- context-history management;
- context compaction/summarisation;
- model request execution;
- provider translation required for request compatibility;
- tool-call continuation semantics;
- heartbeat scheduling;
- progress generation;
- request/session logging;
- orchestration health.

Moderari does not own:

- provider executable lifecycle;
- client UI lifecycle;
- ACP transport;
- tool-provider process lifecycle;
- recording storage;
- replay control;
- assessment;
- whole-stack lifecycle.

---

# Architectural Invariants

## 1. Logical session identity survives transport changes

A conversation does not become a new Lumen session merely because it crosses HTTP, ACP, a tool call, or a recreated UI.

## 2. Moderari owns model-facing context

Clients can display or reconstruct conversations, but the browser is not the authoritative model-context store.

## 3. Provider lifecycle is external

Moderari uses providers. Praebere manages them.

## 4. Protocol bridging is external

Pontis owns ACP and client/provider protocol correlation.

## 5. Backchannel events are operational

Heartbeat and progress information must remain outside conversational content.

## 6. Recording and replay remain independent

Moderari must not absorb Vestigare or Repetere responsibilities.

## 7. Request identity and session identity remain distinct

Many model/tool requests may belong to one logical Lumen session.

## 8. Context management must preserve continuity

Compaction or summarisation may change the representation of context but must not silently change the logical conversation.

---

# Architectural Principle

Moderari is the point at which Lumen understands enough of the active model conversation to orchestrate its continuation.

That understanding must not cause neighbouring responsibilities to migrate into Moderari.

The governing principle is:

> **Moderari owns conversational orchestration and continuity; neighbouring services own transport, provider lifecycle, observability, replay, presentation, and operational control.**

This separation allows models, tool providers, clients, and infrastructure implementations to change without redefining the logical Lumen conversation.
