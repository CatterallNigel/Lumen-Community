# Lumen Audire — Distributed Operational Transport Architecture

**Status:** Architectural Direction / Future Development  
**Date:** 18 August 2026  
**Related Services:** Lumen Servire, Lumen Audire  
**Scope:** Distributed lifecycle management, operational logging and service-state transport

---

## 1. Background

Lumen's services are HTTP-based and are therefore not inherently required to reside on the same physical or virtual host.

A Lumen deployment may initially run entirely on a single machine:

```text
Host A
 ├── Servire
 ├── Moderari
 ├── Pontis
 ├── Praebere
 ├── Rogare
 ├── Vestigare
 ├── Repetere
 ├── Aestimare
 └── Fiducia
```

However, there is no architectural requirement for this arrangement. A future deployment could distribute those services across several hosts:

```text
Host A                    Host B                 Host C
 ├── Servire               ├── Pontis             ├── Fiducia
 ├── Moderari              ├── Vestigare          └── Aestimare
 ├── Praebere              └── Repetere
 └── Rogare
```

The services themselves remain accessible through their HTTP interfaces.

This presents a particular issue for **Lumen Servire** because Servire currently performs more than HTTP service monitoring.

Servire starts and stops local service processes and captures their `stdout` and `stderr` streams to construct its operational logs.

Consequently, the current implementation implicitly assumes that the processes being managed are local child processes of Servire.

A distributed deployment breaks that assumption.

It does **not**, however, require Servire's operational model to change.

The required change is primarily one of **transport**.

---

## 2. Existing Servire Operational Model

Today the relationship can be represented as:

```text
Servire
   │
   ├── starts process
   ├── stops process
   ├── restarts process
   ├── observes process state
   │
   ├── captures stdout
   └── captures stderr
```

For example:

```text
Servire
   │
   ├── Moderari
   │     ├── stdout ──> Servire Operational Log
   │     └── stderr ──> Servire Operational Log
   │
   ├── Pontis
   │     ├── stdout ──> Servire Operational Log
   │     └── stderr ──> Servire Operational Log
   │
   └── Praebere
         ├── stdout ──> Servire Operational Log
         └── stderr ──> Servire Operational Log
```

This provides Servire with both:

1. **Lifecycle control**
2. **Operational visibility**

These capabilities should be preserved in a distributed Lumen deployment.

---

# 3. Architectural Principle

The central architectural decision is:

> **Distributed Lumen should preserve Servire's existing operational semantics while replacing local process communication with a distributed message transport.**

Servire should continue to decide:

- which services start;
- which services stop;
- dependency ordering;
- restart behaviour;
- stack startup and shutdown sequences;
- timeout behaviour;
- failure handling;
- operational status.

The individual Lumen services should not need to become aware of distributed process management.

Instead, process supervision is delegated to a lightweight host service.

That service is:

# Lumen Audire

**Audire** — Latin: *to hear / to listen*

Audire becomes the local operational representative of Servire on each managed host.

---

# 4. Lumen Audire

Audire is intentionally a small service.

It is **not** another orchestration layer.

It does not understand reasoning, sessions, models, Replay behaviour, Trace evidence or Assess results.

Its responsibility is restricted to translating between:

```text
Distributed Operational Messages
            ↕
Local Operating-System Processes
```

Conceptually:

```text
                        Message Broker
                             │
                  ┌──────────┴──────────┐
                  │                     │
               Servire               Servire
                  │
            commands/events
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Audire    Audire    Audire
      Host A    Host B    Host C
        │         │         │
   processes  processes  processes
```

---

# 5. Audire Responsibilities

Audire should have a deliberately narrow responsibility boundary.

## 5.1 Lifecycle Commands

Audire receives lifecycle commands for services resident on its host.

Examples include:

```text
START
STOP
RESTART
```

Potential future commands may include:

```text
STATUS
RELOAD
```

Audire executes the requested operation against the appropriate local process.

---

## 5.2 Process Supervision

Audire starts and supervises local Lumen processes in essentially the same manner that Servire does today.

For example:

```text
Audire
   │
   ├── Moderari
   ├── Praebere
   └── Rogare
```

Audire therefore becomes the parent/supervising process from the perspective of operational management.

---

## 5.3 stdout and stderr Capture

Audire captures:

```text
stdout
stderr
```

from the services it supervises.

The services themselves require no change.

They continue writing operational output exactly as they do today.

Audire packages that output into messages and publishes it to the operational transport.

For example:

```json
{
  "type": "service.output",
  "host": "host-a",
  "service": "moderari",
  "stream": "stdout",
  "timestamp": "2026-08-18T21:14:32Z",
  "message": "Moderari ready"
}
```

or:

```json
{
  "type": "service.output",
  "host": "host-b",
  "service": "pontis",
  "stream": "stderr",
  "timestamp": "2026-08-18T21:15:04Z",
  "message": "Connection reset by peer"
}
```

Servire can consume these messages and construct the same operational view that it provides today.

---

# 6. Distributed Message Transport

A message broker such as **RabbitMQ** is a strong candidate for the operational transport.

RabbitMQ should initially be considered an implementation choice rather than an architectural dependency.

The architectural requirement is:

> **A reliable asynchronous message transport capable of carrying Servire commands and Audire events between hosts.**

The broker sits between Servire and Audire:

```text
                 ┌─────────────────────┐
                 │      Servire        │
                 └──────────┬──────────┘
                            │
                     Commands / Events
                            │
                 ┌──────────▼──────────┐
                 │   Message Broker    │
                 │    e.g. RabbitMQ    │
                 └──────────┬──────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
           Audire         Audire        Audire
           Host A         Host B        Host C
```

---

# 7. Commands and Events

The transport should distinguish between **commands** and **events**.

A command represents something Servire wants to happen.

An event represents something that has happened.

For example:

```text
COMMAND

service.stop
service=Pontis
host=Host-B
command_id=42
```

Audire receives the command and executes it locally.

When the process has stopped:

```text
EVENT

service.stopped
service=Pontis
host=Host-B
command_id=42
```

The correlation identifier allows Servire to associate the event with the command that caused it.

---

# 8. Correlation

Every lifecycle command should carry a unique correlation identifier.

For example:

```json
{
  "type": "service.stop",
  "command_id": "42",
  "host": "host-b",
  "service": "pontis"
}
```

The corresponding acknowledgement contains the same identifier:

```json
{
  "type": "service.stopped",
  "command_id": "42",
  "host": "host-b",
  "service": "pontis",
  "success": true
}
```

This is essential because service-state events can also occur independently.

For example, Pontis could terminate unexpectedly.

Audire might then publish:

```text
service.failed
```

without any corresponding Servire command.

Servire must therefore distinguish between:

```text
requested state transitions
```

and:

```text
unsolicited state transitions
```

---

# 9. Preserving Servire Dependency Ordering

One of the most important consequences of this architecture is that **dependency management remains entirely within Servire**.

Audire does not determine startup or shutdown order.

For example, a stack shutdown might remain:

```text
Servire
   │
   ├── STOP Rogare
   │       │
   │       └── wait for confirmation
   │
   ├── STOP Fiducia
   │       │
   │       └── wait for confirmation
   │
   ├── STOP Moderari
   │       │
   │       └── wait for confirmation
   │
   ├── request model unload through Praebere
   │       │
   │       └── wait for confirmation
   │
   └── STOP Praebere
           │
           └── wait for confirmation
```

The physical locations are irrelevant.

For example:

```text
Rogare       Host A
Fiducia      Host C
Moderari     Host A
Praebere     Host B
```

Servire still sees a logical Lumen stack.

The transport resolves where the commands need to go.

---

# 10. Logical Service Topology vs Physical Topology

This introduces an important architectural distinction.

Servire manages the:

> **Logical Lumen topology**

Audire manages the:

> **Physical host topology**

For example:

```text
LOGICAL

Servire
   │
   ├── Moderari
   ├── Pontis
   ├── Praebere
   ├── Rogare
   └── Fiducia
```

may physically correspond to:

```text
PHYSICAL

NUC
 ├── Audire
 ├── Moderari
 └── Rogare

Zoro
 ├── Audire
 ├── Pontis
 └── Praebere

Zeus
 ├── Audire
 └── Fiducia
```

Servire should not need to change its orchestration behaviour because the physical deployment topology changes.

---

# 11. Operational Logging

The existing Servire operational-log concept can therefore remain intact.

The current path:

```text
Service
   │
stdout/stderr
   │
   ▼
Servire
```

becomes:

```text
Service
   │
stdout/stderr
   │
   ▼
Audire
   │
operational message
   │
   ▼
Message Broker
   │
   ▼
Servire
```

From Servire's perspective, the resulting operational log can remain essentially unchanged.

For example:

```text
21:15:03  NUC    PRAEBERE   Model loaded qwen2.5-coder:14b-32k
21:15:04  NUC    MODERARI   READY
21:15:05  ZORO   PONTIS     Client connection established
21:15:07  NUC    ROGARE     Session started
```

The addition of host identity becomes useful rather than intrusive.

---

# 12. Transport Separation

Operational traffic should not necessarily share a single undifferentiated message stream.

A future RabbitMQ implementation could use logically separate channels such as:

```text
lumen.commands.<host>
lumen.events
lumen.logs
lumen.state
```

For example:

```text
lumen.commands.zoro
    START Pontis
    STOP Vestigare

lumen.commands.nuc
    RESTART Moderari

lumen.events
    service.started
    service.stopped
    service.failed

lumen.logs
    stdout
    stderr

lumen.state
    ready
    degraded
    unavailable
```

The exact exchange and queue topology should be determined during implementation rather than fixed at this architectural stage.

The important principle is that lifecycle control must not become dependent on processing a potentially high-volume stream of operational log messages.

---

# 13. Multiple Servire Instances

A message-based operational transport also removes the assumption that only one Servire instance can observe a deployment.

For example:

```text
                        Message Broker
                       /              \
                      /                \
             Servire Control      Servire Monitor
```

Multiple Servire instances could consume operational events.

However, lifecycle authority should remain explicitly controlled.

A deployment might permit:

```text
Servire-Control
    read events
    issue commands

Servire-Monitor
    read events
    no command authority
```

This should be treated as an access-control concern rather than solved implicitly through queue behaviour.

---

# 14. Failure Behaviour

The distributed architecture introduces failure cases that do not exist in the current local implementation.

Servire must eventually distinguish between:

```text
SERVICE UNAVAILABLE
AUDIRE UNAVAILABLE
HOST UNAVAILABLE
BROKER UNAVAILABLE
COMMAND TIMEOUT
COMMAND FAILED
```

For example:

```text
Servire
   │
   └── STOP Pontis
          │
          ▼
       RabbitMQ
          │
          X
       no Audire response
```

This does **not** necessarily mean Pontis failed to stop.

It means Servire cannot confirm the requested state transition.

The distinction is important.

Therefore:

> **Absence of acknowledgement must not be interpreted as successful completion.**

Servire should preserve its existing wait-for-state behaviour using explicit acknowledgements and timeouts.

---

# 15. Audire Must Not Become Servire

A strict architectural boundary should be maintained.

Audire must not acquire responsibility for:

- dependency ordering;
- stack orchestration;
- deciding whether another service should start;
- deciding recovery strategies;
- reasoning about Lumen state;
- model selection;
- session management;
- Trace control policy;
- Replay control policy;
- Assess interpretation;
- Fiducia decision-making.

Those remain responsibilities of the appropriate Lumen services.

Audire performs requested local operations.

A useful distinction is:

> **Servire decides. Audire listens, acts and reports.**

---

# 16. Relationship to Lumen Evidence

Operational transport must remain separate from Lumen's reasoning evidence architecture.

Audire deals with operational information such as:

```text
Moderari started
Pontis stopped
Praebere exited with code 1
stdout message
stderr message
service READY
service FAILED
```

Vestigare/Trace deals with reasoning and execution evidence such as:

```text
prompts
responses
tool calls
decisions
provenance
execution behaviour
```

These are different concerns.

They may share identifiers where useful for correlation, but Audire must not become an alternative Trace mechanism.

---

# 17. Migration Path

The architecture permits an incremental migration.

## Stage 1 — Current

```text
Servire
   │
local subprocess management
   │
Lumen Services
```

No change required.

## Stage 2 — Audire Prototype

Run Audire on the same host as Servire.

```text
Servire
   │
Message Broker
   │
Audire
   │
local subprocesses
```

This allows the distributed transport to be validated without initially distributing the Lumen services.

## Stage 3 — First Remote Host

Move one service to another machine.

```text
Host A
   Servire
   Audire
   Moderari

Host B
   Audire
   Pontis
```

Servire should continue to operate both services through the same logical control interface.

## Stage 4 — Distributed Lumen

Services can then be distributed according to operational requirements rather than Servire limitations.

```text
             Servire
                │
          Message Broker
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
   Audire    Audire    Audire
   Host A    Host B    Host C
```

---

# 18. Architectural Consequence

This approach preserves one of Lumen's existing architectural strengths:

> **Lumen services are location-independent HTTP services.**

Servire should not undermine that property by requiring all managed services to be local processes.

Equally, distributing Lumen should not require every service to implement a new Servire-specific management interface.

Audire provides the boundary between those concerns.

The resulting architecture becomes:

```text
Application / Reasoning Plane
          │
     Lumen Services
          │
          │ HTTP
          │
────────────────────────────────

Operational Plane
          │
        Audire
          │
   Message Transport
          │
        Servire
```

The application services remain ordinary Lumen HTTP services.

The operational plane becomes independently distributable.

---

# 19. Architectural Decision

The proposed direction is therefore:

> **Lumen Audire will provide lightweight host-level process supervision for distributed Lumen deployments. Audire will capture local service stdout/stderr, execute lifecycle operations requested by Servire, and publish service state and command acknowledgements through a distributed message transport.**

> **Lumen Servire remains responsible for operational orchestration, dependency ordering, lifecycle policy and the overall operational view of the Lumen deployment.**

> **A message broker such as RabbitMQ is the leading candidate for the transport layer, but the architecture should depend on the messaging contract rather than RabbitMQ-specific behaviour.**

The fundamental change from the existing Servire architecture is therefore deliberately small:

> **Servire's operational semantics remain unchanged. The transport between Servire and the processes it manages becomes distributed.**

This allows Lumen to evolve from a single-host deployment to a multi-host deployment without requiring a corresponding redesign of the Lumen services themselves.