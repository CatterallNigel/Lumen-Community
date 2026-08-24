# Lumen Nuntius — Architecture

**Status:** Proposed Architecture  
**Service:** Nuntius  
**Scope:** Lumen `\obt` control-command routing and response distribution

## 1. Architectural Role

Nuntius provides the internal routing layer for the Lumen `\obt` control-command language.

It belongs to the control plane and remains outside the model ask/answer path.

```text
                    CONTROL PLANE

External Client / Rogare
          |
        Pontis
          |
        Nuntius
          |
          +------> command owner
          |
          <------ 200 / 204 / error
          |
          +------> configured response consumers
          |
          +------> Pontis -> originating client


                 MODEL EXECUTION PLANE

External Client
      |
    Pontis
      |
   Moderari
      |
     Model
      |
    Answer
      |
    Trace
```

## 2. Separation of Responsibilities

### Servire

Servire is authoritative for:

- which Lumen services are configured;
- which services are running;
- service endpoints;
- whether a service participates in `\obt`;
- command ownership;
- configured consumers of successful response bodies.

### Nuntius

Nuntius is responsible for:

- obtaining the active control-plane catalogue from Servire;
- materialising an in-memory routing dictionary;
- routing commands to their configured owner;
- correlating command responses;
- returning a terminal outcome for every solicited `\obt` command to its originator;
- distributing successful response bodies to configured Lumen consumers;
- enforcing command timeouts;
- retaining diagnostic evidence for the control-command lifecycle.

### Pontis

Pontis is responsible for:

- recognising client-originated `\obt` traffic;
- forwarding it to Nuntius;
- preserving external session/client correlation;
- returning Nuntius results to the correct originating client.

### Domain Services

Each domain service is responsible for:

- implementing the commands that it owns;
- returning `200`, `204`, or an error;
- returning a response body for query/request commands.

## 3. Servire Control-Plane Catalogue

Nuntius does not infer command ownership.

Servire supplies that information.

A conceptual service definition is:

```yaml
service: praebere
endpoint: http://praebere:11431
obt_enabled: true

commands:
  models:
    response_targets:
      - originator
      - rogare

  providers:
    response_targets:
      - originator
      - rogare

  model_select:
    response_targets:
      - originator
```

The exact configuration representation remains an implementation decision.

The architectural requirement is the information represented, not the syntax.

## 4. Bootstrap

On startup:

```text
Nuntius
   |
   | direct: \obt services
   v
Servire
   |
   | 200 + active control-plane catalogue
   v
Nuntius routing dictionary
```

This is a deliberate bootstrap exception because Nuntius cannot route via a catalogue it has not yet obtained.

Servire may return no active services.

That is healthy if the request itself succeeds.

## 5. Runtime Catalogue Changes

When Servire starts or stops a service, the active control-plane topology changes.

Servire sends an appropriate `\obt` update through Pontis:

```text
Servire
   |
   | \obt service available / unavailable ...
   v
Pontis
   |
   v
Nuntius
   |
   v
routing dictionary updated
```

The update must contain enough information for Nuntius to add, update, or remove the relevant service command routes.

Nuntius may rebuild the complete catalogue at any time by directly requesting:

```text
\obt services
```

from Servire.

Servire remains authoritative.

## 6. Command Routing

The preferred steady-state behaviour is targeted routing, not broadcast-to-all.

For example:

```text
\obt replay list
      |
      v
   Nuntius
      |
      v
  Repetere
```

and:

```text
\obt models
      |
      v
   Nuntius
      |
      v
  Praebere
```

Nuntius therefore requires no model, replay, provider, or prompt domain knowledge. It needs only the routing metadata supplied by Servire.

## 7. Common Service `\obt` Capability

Every Lumen service may expose the same lightweight control endpoint.

A service configuration flag determines whether it participates:

```text
obt_enabled = false
```

If disabled, an incoming `\obt` request returns:

```text
204 No Content
```

immediately.

If enabled, the request proceeds to the service's local `\obt` handler.

An enabled service may still return `204` for an unrecognised or irrelevant command.

This provides a uniform defensive contract even though normal Nuntius routing should target the configured owner directly.

## 8. Response Contract

### 8.1 `204 No Content`

```text
204 No Content
```

Meaning:

> This service did not handle the command.

### 8.2 `200 OK` — No Body

```text
200 OK
```

with no response body means:

> The command was successfully executed and the requested operation has been applied.

For a solicited command, this `200` is returned through Nuntius to the originator as the positive acknowledgement. A caller must never infer success from silence.

Example:

```text
\obt model select <model>
```

### 8.3 `200 OK` — Body Present

```text
200 OK
<body>
```

means:

> The command was a request/query and the body is the authoritative result.

Examples:

```text
\obt replay list
\obt providers
\obt models
\obt services
```

### 8.4 Error

A service that recognises/owns a command but cannot execute it returns an appropriate `4xx` or `5xx`.

Failure must not be represented as `204`.

For a solicited command, the authoritative error is returned through Nuntius to the originator.

### 8.5 Timeout

Nuntius applies a configurable timeout to a solicited command awaiting its authoritative owner response.

If the timeout expires, Nuntius returns:

```text
504 Gateway Timeout
```

to the originator.

A timeout means:

> **The command outcome could not be confirmed within the allowed time.**

It does **not** prove that the owning service did not apply the requested change. The service may have completed the operation but failed to return its response before the timeout.

A caller must therefore treat timeout as an unconfirmed outcome and must not proceed where positive acknowledgement is a prerequisite.

Once Nuntius has returned the timeout to the originator, that request is terminal from the originator's perspective. A later owner response must not be converted into a delayed success response for the completed request. The late response is retained in Nuntius diagnostics.

### 8.6 Solicited Command Completion Rule

Every solicited `\obt` command has one terminal response to its originator:

```text
200, no body
    execution completed successfully

200 + body
    query completed successfully

204
    expected owner did not handle the command

4xx / 5xx
    authoritative command failure

504
    authoritative outcome not confirmed before timeout
```

This rule applies whether the originator is an external client or another Lumen service.

## 9. Returning Outcomes to Originators

Nuntius must preserve request correlation while a solicited command is being handled.

Conceptually:

```text
request_id
session_id
origin
command
```

For an external query:

```text
External Client
    |
    | \obt replay list
    v
Pontis
    |
    | session/request correlation
    v
Nuntius
    |
    v
Repetere
    |
    | 200 + result
    v
Nuntius
    |
    v
Pontis
    |
    | session manager
    v
External Client
```

For a service-originated execution command:

```text
Repetere
    |
    | \obt Moderari: Pass-through
    v
Nuntius
    |
    v
Moderari
    |
    | 200 OK
    v
Nuntius
    |
    v
Repetere
```

Pontis, not Nuntius, owns knowledge of external connection/session identity.

For internal service originators, Nuntius returns the terminal outcome directly through the internal control path using the request correlation.

A `200` with no body is therefore not discarded. It is the positive acknowledgement that allows the originator to know the command completed successfully.

## 10. Shared Response Distribution

Some authoritative query results are needed by more than the originating client.

This sharing policy is defined by Servire configuration.

For example:

```text
\obt models
```

may be configured as:

```text
owner: praebere
response_targets:
    - originator
    - rogare
```

The runtime flow is:

```text
External Client
      |
      v
    Pontis
      |
      v
    Nuntius
      |
      v
   Praebere
      |
      | 200 + models
      v
    Nuntius
     /     \
    /       \
Pontis      Rogare
  |
originator
```

Nuntius does not interpret the model list.

It routes the authoritative response according to the configured targets.

This avoids broadcasting large response bodies to services that do not require them.

## 11. Execution Followed by State Change

A successful execution command may lead the owning service to emit a separate `\obt` command or notification representing the resulting state transition where required.

For example:

```text
\obt model select X
        |
        v
     Praebere
        |
        | 200
        v
     Nuntius
        |
        v
     originator
```

If other services require notification of the resulting selected-model state, that is represented explicitly in the configured control-plane behaviour rather than inferred by Nuntius from the meaning of the command.

Nuntius remains transport and routing infrastructure.

## 12. Trace Boundary

Nuntius-delivered commands do not pass through the normal model ask/answer route.

Therefore they do not become user, assistant, system, or tool turns merely because they were issued.

Execution-affecting results remain observable in Trace.

Examples include:

- selected model;
- selected provider;
- actual system prompt reaching the model;
- other execution conditions required for reproduction.

### Replay System Prompt

Repetere must replay the effective system prompt captured in the source Trace.

Before replay begins, Repetere uses `\obt` through Nuntius to instruct Moderari to use Pass-through for the replay session.

The override must be session-scoped.

This is a blocking prerequisite. Repetere must receive Moderari's authoritative `200 OK` through Nuntius before any replay model interaction begins.

For this prerequisite:

```text
200
    proceed

204
    abort — expected owner did not handle the command

4xx / 5xx
    abort — requested state was not successfully established

504 / timeout
    abort — outcome cannot be confirmed
```

A failed prerequisite leaves the corresponding replay/Experiment run `FAILED / INCOMPLETE`.

The `\obt` command itself is not written into the conversational Trace; the actual original system prompt supplied to the model is.

## 13. Observability and Logging Boundaries

Nuntius requires its own control-plane diagnostic evidence.

That evidence must not be mixed into the default Servire operational log and must not appear in Vestigare conversational Trace.

The architectural boundary is:

> **Servire Ops log = what matters operationally to the Lumen operator.**  
> **Nuntius diagnostics = what happened inside the control-command transport.**  
> **Vestigare Trace = what happened in the model execution.**

### 13.1 Nuntius Diagnostic Record

For each solicited command, Nuntius should retain enough information to reconstruct the command lifecycle, including where available:

- timestamp;
- request ID;
- origin/originator;
- originating session identifier where relevant;
- command;
- resolved command owner;
- configured response targets;
- dispatch time;
- owner response status;
- elapsed time;
- timeout;
- result-routing outcome;
- routing/configuration failure;
- late owner response after timeout.

Diagnostic records are control-plane evidence. They are not conversational evidence.

### 13.2 Minimal Nuntius Diagnostics UI

M0.1 should provide a lightweight Nuntius diagnostics view sufficient for development and research-distribution support.

At minimum it should expose:

```text
Health
Servire connectivity
active route count
in-flight command count
recent command lifecycle records
```

A recent-command row should make it possible to distinguish, for example:

```text
completed
rejected
owner error
not handled
timed out / outcome unconfirmed
late response
routing failure
```

Selecting or expanding a record should expose its request correlation and routing details.

### 13.3 Diagnostic Retention

M0.1 does not require Nuntius to become a durable audit store.

A bounded diagnostic history is sufficient initially. The retention mechanism and size should be configurable without introducing RabbitMQ, Kafka, or another persistent messaging dependency.

### 13.4 Servire Operational Log

The existing problem whereby Rogare polling and internal maintenance activity can obscure meaningful user actions remains a separate Servire/Rogare concern.

A future `log_class` or equivalent may support user/internal filtering, but it is explicitly deferred from the initial Nuntius routing contract.

Nuntius diagnostics must not be copied wholesale into the Servire Ops log merely to make them visible.

## 14. Failure Boundaries

### Servire unavailable during bootstrap

Unhealthy Nuntius startup.

### Servire reachable with no active services

Healthy.

### Configured owner unavailable

Nuntius returns an appropriate routing/delivery failure and does not silently substitute another service.

### Unknown command

If the routing dictionary contains no owner for the command, Nuntius reports that the command is not available in the active topology.

### Owner timeout

Nuntius returns `504 Gateway Timeout` to the originator and records the command outcome as unconfirmed.

A late owner response is retained diagnostically but does not change the already-terminal response delivered to the originator.

### Nuntius unavailable

`\obt` control operations that depend on Nuntius cannot be delivered.

Normal model ask/answer traffic should not depend on Nuntius unless a specific operation requires a control-plane change.

## 15. Architectural Principle

> **Servire defines who owns each active control command and who needs its response. Nuntius materialises that configuration into a runtime routing table and transports commands and responses without acquiring domain knowledge.**
