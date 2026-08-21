# Lumen Moderari Overview

## Overview

Lumen Moderari is the orchestration and continuity service within the Lumen stack.

Its purpose is to sit between the communication/observability layers and the configured model provider, preserving the logical conversation while coordinating model requests, tool-use continuations, context management, and operational progress.

Moderari is deliberately model-provider independent at its architectural boundary. Provider lifecycle and model availability belong to Praebere; communication/session bridging belongs to Pontis; recording and replay belong to Vestigare and Repetere.

Moderari's principal responsibility is:

**Orchestrate the model-facing conversation while preserving Lumen session continuity and behavioural context.**

---

## Position in the Lumen Stack

A normal Rogare conversation follows the broad path:

```text
Rogare
   |
   v
Pontis
   |
   v
Vestigare
   |
   v
Repetere
   |
   v
Moderari
   |
   v
Model provider
```

When tools are required, Pontis provides the bridge to the configured tool provider, currently Pi, while Moderari remains responsible for the model-facing conversational flow.

Praebere operates alongside this path as the provider lifecycle service:

```text
                 Servire
                    |
        +-----------+-----------+
        |                       |
        v                       v
     Moderari                Praebere
        |                       |
        | model requests        | provider lifecycle
        +-----------+-----------+
                    |
                    v
              Ollama / Model
```

---

## Primary Responsibilities

Moderari owns the orchestration concerns associated with an active Lumen conversation.

These include:

- accepting OpenAI-compatible model requests;
- maintaining logical Lumen session identity;
- preserving and managing conversation context;
- coordinating model request/response cycles;
- supporting tool-call continuation through the surrounding Lumen stack;
- applying context-management and compaction behaviour;
- producing heartbeat and progress information during long-running requests;
- exposing operational information needed by Servire;
- maintaining model-facing behavioural continuity independently of the client UI.

Moderari does not own the user interface, provider process lifecycle, protocol bridging, recording, replay, or assessment.

---

## Session and Context Continuity

The logical Lumen session is the continuity boundary for Moderari.

For Rogare-originated conversations this normally takes the form:

```text
session-rogare-YYYYMMDD-HHMMSS
```

The session identity must survive:

- multiple user requests;
- model responses;
- tool calls and tool-result continuations;
- Rogare embedded/pop-out transitions;
- ACP provider activity bridged through Pontis.

Moderari is the authoritative model-facing holder of conversational context. Rogare may reconstruct the visible conversation when its UI is recreated, but browser presentation state is not the authoritative source of model context.

Pontis is responsible for preserving the relationship between the logical Lumen session and any separate ACP/Pi session. Moderari should not need to understand ACP session identities.

---

## Model Interaction

Moderari sends model requests to the configured provider endpoint.

The provider itself is not owned by Moderari.

For the current local deployment:

```text
Moderari
    |
    | OpenAI-compatible model request
    v
Ollama
    |
    v
Qwen
```

Praebere is responsible for determining and managing the provider/model lifecycle. This keeps orchestration separate from infrastructure management.

Moderari may know which supplier and model are required for a request, but starting, stopping, loading, unloading, and checking provider readiness are Praebere responsibilities.

---

## Tool-Use Flow

Moderari participates in tool-capable conversations without becoming the tool-provider bridge.

The broad flow is:

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
   | tool request
   v
Pontis <----> Pi
   |
   | continuation/tool result
   v
Moderari
   |
   v
Model
```

The exact transport may pass through Vestigare and Repetere according to the active Lumen configuration.

Pontis owns the communication relationship with Pi. Moderari owns the model-facing continuation of the logical conversation.

---

## Context Management

Moderari is responsible for keeping model context usable across a continuing session.

This includes the established Lumen context-management behaviours such as:

- tracking conversation history;
- monitoring context consumption;
- performing configured compaction/summarisation behaviour;
- preserving continuity across context-management operations;
- maintaining the session identity while context representation evolves.

Context management is an orchestration concern rather than a client concern.

A client such as Rogare should not need to recreate or resend the authoritative conversational state merely because its own UI has been reloaded.

---

## Heartbeat and Progress

Long-running model requests can make a synchronous client appear inactive even though useful work is continuing.

Moderari therefore generates backchannel events during long operations.

These include:

- heartbeat events;
- progress events;
- model-related progress information;
- elapsed processing information;
- estimated context utilisation where available.

The events are associated with the logical Lumen session and delivered to Pontis.

Rogare polls the Pontis session backchannel and presents the resulting liveness/progress information to the operator.

Conceptually:

```text
Moderari
   |
   | heartbeat / progress
   v
Pontis backchannel
   |
   | polling
   v
Rogare
```

The heartbeat is an operational liveness signal. It is not conversational content and should not become part of the model dialogue.

---

## Relationship to Pontis

Pontis is the communication bridge.

Moderari is the orchestrator.

Pontis owns:

- HTTP/ACP bridging;
- ACP session management;
- tool-provider communication;
- correlation between Lumen sessions and ACP sessions.

Moderari owns:

- model-facing conversation orchestration;
- logical conversational context;
- model request continuity;
- context management;
- model-processing heartbeat/progress generation.

This separation prevents Moderari from becoming dependent on Pi, ACP, Rogare, or any particular client protocol.

---

## Relationship to Praebere

Praebere is the provider lifecycle service.

Moderari may identify the supplier/model required by the conversation, but Praebere owns the operational provider state.

Praebere is responsible for:

- starting the configured provider;
- stopping a provider that it owns;
- model loading/warm-up;
- model unloading;
- provider readiness and availability;
- reporting provider/model state.

Moderari is responsible for using the provider once it is available.

This creates a deliberate boundary:

> Moderari orchestrates model use; Praebere manages model availability.

---

## Relationship to Vestigare and Repetere

Vestigare records observable Lumen traffic and provenance.

Repetere controls replay behaviour.

Moderari does not absorb either responsibility.

Normal traffic may pass through these components before reaching Moderari, but Moderari remains concerned with the live model-facing conversation rather than recording or experimental replay control.

---

## Relationship to Rogare

Rogare is the conversational client.

It presents:

- session controls;
- user messages;
- model responses;
- heartbeat/progress information;
- operator-visible session state.

Moderari does not own Rogare's presentation state.

Conversely, Rogare does not own Moderari's authoritative conversation context.

This separation allows a Rogare UI to be destroyed and recreated while the logical Lumen session continues.

---

## Relationship to Servire

Servire is the operational control surface for the Lumen stack.

Servire starts, monitors, displays, and stops Moderari as a managed component.

Moderari exposes operational information but does not become responsible for stack lifecycle management.

---

## Operational Characteristics

Moderari is designed for potentially long-running requests.

Important runtime characteristics include:

- long model-request timeouts;
- heartbeat generation while model work continues;
- progress reporting independent of final responses;
- preservation of session identity through tool continuations;
- context-management decisions during extended conversations;
- transparent participation in the wider Lumen observability/replay path.

A slow model response is therefore not automatically equivalent to a stalled Moderari request.

---

## Architectural Boundary

Moderari should understand the logical model conversation.

It should not need to understand:

- how Rogare is embedded in Servire;
- how Pi implements ACP;
- how Pontis maps ACP session identifiers;
- how Ollama is launched or terminated;
- how Vestigare persists traces;
- how Repetere evaluates replay matching;
- how future Assess/Fiducia services judge or schedule behaviour.

Keeping these responsibilities outside Moderari is essential to maintaining Lumen's modular architecture.

---

## Summary

Lumen Moderari is the model-facing orchestration and continuity layer of Lumen.

It provides the stable conversational boundary between clients/tools and interchangeable model providers while maintaining session identity, context, request continuity, and operational progress.

Its defining separation is:

> **Moderari decides how the active Lumen conversation is carried forward. It does not own the client, tool provider, model process, recorder, replay controller, or operational control plane.**
