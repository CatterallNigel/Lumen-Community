# Lumen Pontis

## Client Session Bridge

**Version:** 1.0  
**Status:** Design  
**Component:** Lumen Extension

---

# Overview

Lumen Pontis is the client session bridge for the Lumen ecosystem.

It provides a stable abstraction layer between external AI clients and the internal Lumen behavioural engineering stack, allowing clients, protocols and tool providers to evolve independently from Trace, Replay, Assess and Lumen.

Pontis is intentionally lightweight. Its responsibility is to faithfully bridge conversations between clients and the Lumen ecosystem while providing the client-specific functionality required for advanced session operations.

Pontis has no behavioural engineering responsibilities.

---

# Why Pontis Exists

As Lumen has evolved, responsibilities have become increasingly well separated:

- **Lumen** orchestrates models, tools and continuity.
- **Trace** observes and records behaviour.
- **Replay** reproduces recorded behaviour.
- **Assess** evaluates behavioural quality.
- **Servire** manages lifecycle and system orchestration.

What remained was a component responsible for understanding client protocols and session management.

Rather than allowing Trace or Replay to become aware of client-specific behaviour, Pontis provides a dedicated client bridge that isolates these concerns.

---

# Design Goals

Pontis has four primary goals:

1. Keep the behavioural engineering stack client-agnostic.
2. Provide transparent message forwarding during normal operation.
3. Enable advanced session operations without modifying clients.
4. Support multiple client protocols through a common abstraction.

---

# Position within the Lumen Stack

```
           Client
              │
              ▼
      +---------------+
      |    Pontis     |
      +---------------+
              │
              ▼
      +---------------+
      |    Trace      |
      +---------------+
              │
              ▼
      +---------------+
      |    Replay     |
      +---------------+
              │
              ▼
      +---------------+
      |    Lumen      |
      +---------------+
              │
              ▼
      Model / Tools
```

Pontis sits north of Trace and is the first Lumen component encountered by client traffic.

---

# Responsibilities

Pontis is responsible for:

- Client protocol integration
- Session management
- Client identity
- Message forwarding
- Session routing
- Capability negotiation
- Client reconnection
- Session migration
- Replay reinjection
- Tool provider bridging

---

# Non-Responsibilities

Pontis deliberately does **not**:

- record conversations
- replay conversations
- compare behaviour
- analyse responses
- score quality
- manage checkpoints
- own continuity
- orchestrate models
- perform behavioural assessment

Those responsibilities belong to Trace, Replay, Assess or Lumen.

---

# Normal Operation

During normal conversations Pontis behaves as a transparent bridge.

```
Client
   │
Pontis
   │
Trace
   │
Replay (transparent)
   │
Lumen
   │
Model
```

Messages pass through unchanged.

Replay, Trace and Lumen remain unaware that Pontis exists.

---

# Advanced Operation

Pontis is always the client session bridge.

During normal operation it behaves as a transparent passthrough.

During advanced operations—such as Replay continuation, session migration, client reconnection, or future cross-client handoff—it becomes an active session mediator.

This activation occurs without altering the responsibilities of Trace, Replay or Lumen.

---

# Replay Integration

Replay is responsible for behavioural reproduction.

Pontis is responsible for client interaction.

When Replay reaches a behavioural fork requiring continuation inside a live client session, Replay delegates reinjection to Pontis.

Replay remains completely unaware of client protocols.

Pontis understands how to resume the conversation for the connected client.

---

# Client Adapters

Pontis is designed around client adapters.

Initial support will include:

- Pi

Future investigation includes:

- ACP
- Zed ACP
- VS Code
- Cursor
- Claude Desktop
- OpenAI Responses API
- Anthropic APIs

Additional adapters should require no changes to Trace, Replay or Lumen.

---

# Tool Provider Integration

Pontis may also become the abstraction layer for tool providers.

This enables client-specific tool behaviour to remain isolated from the behavioural engineering stack.

Tool providers remain independent of Trace, Replay and Assess.

---

# Design Principles

Pontis follows several architectural principles.

## Single Responsibility

Pontis bridges sessions.

Nothing more.

## Transparency

Normal conversations should behave exactly as if Pontis were not present.

## Client Isolation

Client-specific behaviour must never leak into Trace, Replay or Lumen.

## Extensibility

New clients should be introduced by adding adapters rather than modifying existing components.

---

# Future Possibilities

Pontis provides the architectural foundation for future capabilities including:

- session migration
- reconnect after client failure
- multi-client continuity
- client failover
- protocol translation
- live conversation transfer
- shared session routing

These capabilities can be introduced without changing the behavioural engineering stack.

---

# Summary

Pontis completes the separation between client communication and behavioural engineering.

By isolating client protocols into a dedicated bridge, Trace, Replay, Assess and Lumen remain entirely focused on their own responsibilities while Pontis provides a stable integration layer for present and future AI clients.

---

# Architectural Invariant

Pontis is a **bridge**, not an inference component.

Its sole responsibility is to bridge client sessions and protocols into the Lumen ecosystem while remaining transparent during normal operation.

Like every component within the **Lumen++** architecture, Pontis adheres to the principle of **Single Responsibility**. It exists to manage client communication and session mediation—nothing more.

## Standalone Development

During standalone development and validation, Pontis may terminate incoming client requests and return deterministic responses.

This capability exists **only** to verify communication paths, protocol implementations and session handling independently of the remainder of the Lumen stack.

These responses are test fixtures and must never be considered part of Pontis' operational behaviour.

## Normal Operation

In normal Lumen operation, Pontis does **not** generate conversational responses.

Instead, OpenAI-compatible client traffic is forwarded transparently into the downstream Lumen stack without modification.

Under normal operation Pontis:

- Accepts client requests.
- Maintains client sessions.
- Routes requests into the Lumen stack.
- Returns downstream responses to the originating client.

Pontis does **not**:

- Generate AI responses.
- Invoke language models.
- Record conversations.
- Replay conversations.
- Assess behaviour.
- Maintain continuity.
- Modify prompts or responses.
- Make behavioural decisions.

These responsibilities belong exclusively to other Lumen components.

## Active Session Mediation

Although normally transparent, Pontis may become an active session mediator when requested by another Lumen component.

Examples include:

- Replay continuation following a behavioural fork.
- Session migration.
- Client reconnection.
- Future cross-client handoff.

Even during these operations, Pontis remains responsible only for client session management and protocol translation. The decision to perform these actions originates elsewhere within the Lumen ecosystem.

## Architectural Principle

The behaviour of Pontis can be summarised by a simple rule:

> **During development, Pontis may terminate requests for testing. During normal operation, Pontis never becomes the conversational endpoint. It transparently bridges client communication into and out of the Lumen stack.**

This invariant preserves the architectural integrity of the Lumen++ ecosystem by ensuring that every component retains a single, clearly defined responsibility.