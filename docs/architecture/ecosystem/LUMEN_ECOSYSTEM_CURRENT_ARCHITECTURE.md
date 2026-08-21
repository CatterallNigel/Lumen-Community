# Lumen Ecosystem — Current Architecture and Topology

**Status:** Canonical  
**Document role:** Current ecosystem architecture  
**Last reviewed:** 2026-08-21

---

## Purpose

This document is the canonical high-level description of the **current Lumen service topology**.

It answers four questions:

1. What services currently make up Lumen?
2. What is each service responsible for?
3. How do requests, evidence and operational control move through the system?
4. Which parts are implemented today and which remain architectural direction?

Detailed behaviour remains defined by the architecture documents within each service folder. This document describes the relationships between them.

---

## Architectural Principle

> **Lumen is the product; the named components are independently bounded Lumen services.**

Each service should have one primary job. Services may cooperate, but responsibility should not leak across service boundaries merely for implementation convenience.

The current ecosystem comprises:

- **Moderari** — orchestration
- **Pontis** — bridge and protocol/session correlation
- **Praebere** — model-provider abstraction and lifecycle
- **Vestigare** — trace/evidence capture
- **Repetere** — replay and divergence reproduction
- **Aestimare** — behavioural assessment
- **Fiducia** — repeatable Replay orchestration
- **Rogare** — conversational/operations client
- **Servire** — operational control plane
- **Audire** — proposed distributed operational transport

---

## Current Runtime Topology

The principal conversational and evidence path is:

```text
User
  |
  v
Rogare :11432
  |
  | conversational / provider interaction
  v
Pontis :11435
  |
  | OpenAI-compatible HTTP, transparently forwarded
  v
Vestigare :11438
  |
  | recorded request/response traffic
  v
Repetere :11437
  |
  | transparent during normal execution;
  | controlled replay when explicitly invoked
  v
Moderari :11436
  |
  | model-request orchestration
  v
Model provider
  |
  +--> currently Ollama :11434

Praebere :11431
  +--> provider lifecycle, availability and model state
  +--> used by Servire and the Moderari/provider boundary

Fiducia :11430
  +--> schedules and orchestrates repeated Repetere executions
  +--> creates populations of Replay evidence for later assessment

Servire :11439
  +--> dependency validation
  +--> service lifecycle
  +--> operational state and logs
  +--> unified access to service operational interfaces

Aestimare
  +--> assessment layer under active design/development
  +--> consumes retained behavioural evidence rather than sitting in the live request path

Audire
  +--> future distributed operational transport
  +--> intended to decouple Servire lifecycle/log transport from local process ownership
```

Ports shown are the current canonical local development ports where established. They are deployment configuration, not architectural identity.

---

## Communication Planes

Lumen currently contains several distinct communication concerns. They should remain conceptually separate.

### Conversational request plane

The primary model-request path uses OpenAI-compatible HTTP.

Pontis forwards HTTP traffic opaquely into the downstream Lumen stack. Vestigare observes and records that traffic. Repetere remains transparent until replay behaviour is explicitly required. Moderari owns model-request orchestration.

### Tool/provider control plane

Pontis also supports ACP relationships with external tool providers such as Pi. ACP session identity is distinct from the originating Lumen conversational session and is correlated by Pontis rather than exposed to Rogare.

Moderari may require external capabilities during orchestration, but provider-specific vocabulary should remain behind the appropriate Lumen boundary.

### Model-provider plane

Praebere abstracts external model-provider lifecycle and model state.

Praebere is responsible for provider-specific operations such as detecting availability, establishing model readiness and respecting managed-versus-external provider ownership. Moderari remains responsible for deciding when model capability is required; provider-specific lifecycle details belong to Praebere.

### Evidence plane

Vestigare records execution evidence. Repetere reproduces an existing execution and identifies divergence. Fiducia schedules repeated Replay activity. Aestimare interprets the resulting evidence.

This produces a deliberate progression:

```text
Execution
   |
   v
Vestigare records
   |
   v
Repetere reproduces
   |
   v
Fiducia repeats under controlled scheduling
   |
   v
Aestimare assesses populations of evidence
```

A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows behavioural change to be detected.

### Operational control plane

Servire is responsible for operating Lumen services, not for owning their domain behaviour.

Servire validates dependencies, starts and stops managed components in dependency order, exposes operational state, aggregates operational logs and provides navigation to service-owned interfaces.

The current local implementation can own child processes and consume their stdout/stderr directly. Audire is the architectural direction for retaining the same Servire responsibilities when services are distributed across hosts.

---

## Service Responsibility Boundaries

### Moderari — Orchestration Service

Moderari owns model-request orchestration and context/policy intervention required by Lumen. It should not absorb provider lifecycle, client UI, trace persistence, replay analysis or operational process management.

### Pontis — Bridge Service

Pontis bridges clients, Lumen's HTTP request path and external ACP/tool-provider relationships. It owns protocol/session correlation but treats the model-request HTTP payload as opaque.

### Praebere — Integration / Model Provider Service

Praebere provides the Lumen-side abstraction over external model providers. It owns provider lifecycle and model readiness semantics while preventing Ollama-specific or other provider-specific behaviour leaking into the rest of Lumen.

### Vestigare — Trace Service

Vestigare transparently records model-request traffic and persists execution evidence. It records what occurred; it does not decide whether the behaviour was good, correct or acceptable.

### Repetere — Replay Service

Repetere reproduces recorded executions under controlled conditions, detects the first divergence and then allows the new execution to continue transparently. Its job is reproduction, not assessment.

### Fiducia — Reasoning Assurance Orchestration Service

Fiducia decides **when, how often and how many Replay runs should occur**. It schedules repeated executions and retains their relationship to the originating Prepared Replay. It does not determine what the evidence means.

### Aestimare — Assess Service

Aestimare determines what accumulated behavioural evidence means. Its emerging architecture is modular: specialist internal assessment engines examine narrowly defined dimensions and Aestimare synthesises their evidence. Aestimare is not part of the synchronous inference path.

### Rogare — Operations / Conversational Client

Rogare provides the human conversational and operational interaction surface. It uses Lumen services rather than reimplementing their internal responsibilities and does not need to understand provider-specific ACP mechanics.

### Servire — Operational Control Plane

Servire owns operational lifecycle, dependency validation, state, configuration presentation and service log aggregation. It does not implement the domain logic of the services it manages.

### Audire — Distributed Operational Transport

Audire is currently architectural direction rather than an implemented core service. It is intended to supervise a service process locally, capture stdout/stderr, receive lifecycle commands and exchange operational messages through a broker such as RabbitMQ so that Servire can retain its control-plane role across distributed hosts.

---

## Current Managed Startup Dependency Direction

The current Servire-managed sequence is broadly:

```text
Praebere
   |
Moderari
   |
Repetere
   |
Vestigare
   |
Pontis
   |
Rogare
   |
Fiducia
```

This is an operational ordering, not the request-flow direction.

Fiducia starts after the Replay execution path because its scheduler becomes active when the service starts. It currently depends on Repetere, Pontis and MongoDB.

External dependencies such as MongoDB, Redis and the configured model provider are validated according to the services that require them.

---

## Current versus Future Architecture

### Implemented / operational

The current engineering evidence establishes operational implementations of:

- Moderari
- Pontis
- Praebere
- Vestigare
- Repetere
- Rogare
- Servire
- Fiducia

These services have reached different maturity levels, but each has executable implementation and documented integration evidence.

### Active development

**Aestimare** is the current major development direction. Its role, experimental methodology and modular assessment architecture are documented, but it should not yet be described as an operational assessment capability equivalent to the implemented services above.

### Architectural direction

**Audire** is a proposed distributed operational transport architecture. It should be described as future development until implementation evidence exists.

---

## Primary Data and State Boundaries

### MongoDB

MongoDB is used by Lumen services that persist trace, replay, continuity or orchestration evidence. Ownership of individual collections belongs to the relevant service; MongoDB itself is an infrastructure dependency rather than a Lumen service.

### Model provider

The model provider is external to Lumen. Ollama is the current principal implementation used in development, but Lumen architecture must remain provider-neutral.

### Tool providers

External tool providers are also outside the Lumen service boundary. Pi is an important current implementation and architectural test case, not the definition of the tool-provider contract.

---

## Canonical Reading Path

For the ecosystem as a whole:

1. This document — current topology and responsibility boundaries.
2. [`LUMEN_SERVICE_MATRIX.md`](LUMEN_SERVICE_MATRIX.md) — concise service responsibilities, dependencies, interfaces and maturity.
3. Individual service `README.md` files under `docs/services/`.
4. The canonical/current architecture document for the service in question.
5. `version-history/` only when historical milestone or implementation provenance is required.

For extension boundaries, see:

- [`../extensions/LUMEN_THIRD_PARTY_EXTENSION_ARCHITECTURE.md`](../extensions/LUMEN_THIRD_PARTY_EXTENSION_ARCHITECTURE.md)

For installation/distribution direction, see:

- [`../LUMEN_INSTALLATION_AND_DISTRIBUTION_ARCHITECTURE.md`](../LUMEN_INSTALLATION_AND_DISTRIBUTION_ARCHITECTURE.md)

---

## Architectural Invariants

The following rules should remain true as implementation evolves:

1. **One job first.** Each service owns a bounded responsibility.
2. **Observe, reproduce, repeat, assess are separate concerns.**
3. **Operational control is not behavioural control.** Servire operates services; it does not absorb their domain behaviour.
4. **Provider-specific semantics stay at provider boundaries.**
5. **Evidence is retained before it is interpreted.**
6. **Historical implementation detail must not silently redefine current architecture.**
7. **Future capability must be labelled as future capability.**
8. **The service topology may be distributed without changing the logical responsibility model.**

---

## Maintenance Rule

When a service responsibility, canonical port, primary dependency or request-path relationship changes, this document and the service matrix should be updated in the same change as the canonical service architecture.

Historical milestone documents should not be rewritten to match the new topology; their value is that they preserve what was true at the time.
