# Lumen Pontis Architecture

## Overview

Pontis is the communication bridge between external AI clients and the Lumen stack.

It has a single responsibility:

**Bridge communication protocols without understanding conversational content.**

Pontis deliberately avoids any behavioural interpretation of model traffic.

Conversation understanding remains entirely within Lumen.

---

# Architectural Position

```
                    Servire
                        │
                        │
                Management API
                        │
                        ▼
                  Lumen Pontis
               ┌────────┴────────┐
               │                 │
              ACP              HTTP
               │                 │
               ▼                 ▼
              Pi        Trace → Replay → Lumen
                                         │
                                         ▼
                                      Model
```

Pontis forms the protocol boundary between Pi and the remainder of the Lumen stack.

---

# Dual Communication Planes

Pontis manages two independent communication planes.

## ACP Plane

Responsible for:

- starting Pi conversations
- creating ACP sessions
- maintaining session mappings
- receiving conversational updates
- receiving tool status
- receiving final responses

ACP is the control plane.

It is not the model communication protocol.

---

## HTTP Plane

Responsible for:

- model prompts
- streaming responses
- tool calls
- tool results
- continuation requests

Pontis treats HTTP traffic as completely opaque.

Requests are forwarded without inspection.

Responses are forwarded without modification.

Conversation understanding belongs entirely to Lumen.

---

# Conversation Lifecycle

## Pi-originated conversations

```
Pi
 │
 ▼
HTTP
 │
 ▼
Pontis
 │
 ▼
Trace
 │
 ▼
Replay
 │
 ▼
Lumen
 │
 ▼
Model
```

No ACP involvement exists.

---

## Servire-originated conversations

```
Servire
 │
 ▼
Pontis
 │
 │ ACP
 ▼
Pi
 │
 │ HTTP
 ▼
Pontis
 │
 ▼
Trace
 │
 ▼
Replay
 │
 ▼
Lumen
 │
 ▼
Model
```

The conversation begins using ACP.

Once Pi starts reasoning it immediately communicates with the model using HTTP.

The final conversational output returns to Servire over ACP.

---

# Session Management

Pontis maintains a mapping between:

- Servire/Lumen session IDs
- ACP session IDs

The mapping exists solely for routing.

Pontis does not interpret conversation history.

---

# Transparency Principle

Pontis is intentionally transparent.

It:

- does not parse prompts
- does not parse responses
- does not understand tool calls
- does not understand OpenAI protocol semantics
- does not understand Pi behaviour

All conversational semantics belong to Lumen.

---

# Runtime Responsibilities

Pontis owns:

- ACP runtime
- Pi lifecycle
- HTTP proxy
- session mappings
- operational state
- logging
- runtime health

Pontis does not own:

- conversation understanding
- behavioural analysis
- replay
- assessment
- provenance
- checkpoints

These remain Lumen responsibilities.

---

# Architectural Invariant

Pontis may terminate requests during standalone development and testing.

During normal Lumen operation Pontis does not generate conversational responses.

Pontis transparently forwards OpenAI-compatible HTTP traffic into the downstream Lumen stack while independently managing ACP communication with Pi.

Every component within the Lumen++ architecture maintains exactly one responsibility.

This invariant preserves the separation between protocol management and behavioural understanding.

---

# Relationship to Servire

Servire is the operational control surface.

Pontis is the communication bridge.

Servire starts, monitors and manages Pontis.

Pontis manages Pi.

Lumen manages conversation behaviour.

Each component remains independent while collaborating through clearly defined interfaces.

This separation allows future client protocols to be added without altering Lumen itself.