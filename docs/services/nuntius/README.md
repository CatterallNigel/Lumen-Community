# Lumen Nuntius

**Nuntius** is Lumen's lightweight control-command distribution service.

Its purpose is to establish `\obt` as a common Lumen control-command language and distribute those commands across the Lumen services currently available to receive them.

Nuntius is intentionally small. It does not execute domain-specific commands, manage service lifecycle, participate in model conversations, or provide durable messaging.

> **Given a Lumen control command and the services currently available, Nuntius delivers the command without requiring the sender to understand the Lumen service topology.**

## Role in Lumen

Conceptually:

```text
External Client / Rogare / Lumen Service
                  |
                Pontis
                  |
             \obt command
                  |
               Nuntius
             /    |     \
            /     |      \
     Moderari   Replay   Praebere ...
```

Pontis recognises `\obt` control traffic and forwards it to Nuntius.

Nuntius maintains a catalogue of currently available Lumen services and distributes the command to them.

Each receiving service decides whether the command is relevant to its responsibilities:

- `204 No Content` — command is not relevant to the service.
- `200 OK` — command was recognised and successfully handled.
- `4xx/5xx` — command was recognised or assigned but could not be successfully executed.

## Service Discovery

Servire is authoritative for the active Lumen service topology.

When Nuntius starts, it contacts Servire directly using:

```text
\obt services
```

Servire may return a populated catalogue or no active services.

An empty catalogue is a valid and healthy Nuntius startup state. The important startup condition is that Nuntius can communicate with Servire and obtain Servire's current view of the stack.

As Servire starts and stops services, service-state changes are emitted as `\obt` commands through Pontis and forwarded to Nuntius so that its local catalogue remains current.

## Trace Boundary

Nuntius operates on Lumen's control plane rather than the normal model ask/answer path.

Internal `\obt` traffic must therefore not appear as conversational turns in Trace.

Where a control command changes the conditions of a subsequent model execution, Trace records the resulting execution conditions rather than the internal command used to establish them.

For example, model/provider selection and the actual injected system prompt remain part of the execution evidence required for Replay and assessment.

## Documentation

This folder contains the current Nuntius design documentation:

- **[NUNTIUS_OVERVIEW.md](NUNTIUS_OVERVIEW.md)** — purpose, responsibilities, boundaries and relationship to the wider Lumen architecture.
- **[NUNTIUS_ARCHITECTURE.md](NUNTIUS_ARCHITECTURE.md)** — command flow, Servire bootstrap, service catalogue, response semantics, Trace boundary and service responsibilities.
- **[NUNTIUS_DEVELOPMENT.md](NUNTIUS_DEVELOPMENT.md)** — proposed implementation sequence, investigation of existing `\obt` behaviour, integration requirements and testing.

## Current Status

Nuntius is currently a proposed Lumen service.

Before implementation, the existing `\obt` mechanisms in Replay, Moderari and Pontis should be examined so that Nuntius consolidates and generalises existing behaviour rather than introducing another command mechanism.

Praebere provider/model discovery and selection is expected to be one of the first new capabilities to use the common Nuntius path.

## Architectural Boundary

Nuntius is **not**:

- a service orchestrator;
- a replacement for Servire;
- a durable message broker;
- a tool catalogue;
- a model-provider abstraction;
- part of the conversational execution path.

Its responsibility is command distribution.

Servire remains authoritative for operational service state, Pontis remains the external client bridge, and individual Lumen services remain responsible for interpreting and executing the commands that belong to them.
