# Lumen Nuntius — Development Plan

**Status:** Proposed Development  
**Service:** Nuntius  
**M0.1 Priority:** Foundational  
**Dependency:** Existing Replay, Moderari and Pontis `\obt` behaviour must be inspected before implementation.

## 1. Development Objective

Implement the shared Lumen control-plane routing service before extending the M0.1 services that consume it.

Nuntius is now foundational M0.1 infrastructure because Replay system-prompt fidelity, Praebere discovery/selection, and Servire topology updates all require a common `\obt` path.

## 2. Pre-Implementation Investigation

Inspect the current `\obt` implementation in:

- Repetere;
- Moderari;
- Pontis.

Establish:

- current command syntax;
- how commands are detected;
- how Replay currently injects commands;
- how Moderari currently injects commands;
- how command responses are currently propagated;
- whether commands enter the normal ask/answer path;
- how session/request correlation currently works;
- what tests already exist.

The implementation should generalise existing behaviour rather than create a competing mechanism.

## 3. Servire Configuration Dependency

Before Nuntius routing is complete, Servire must be able to describe the active control-plane topology.

Required metadata includes:

- service identity;
- service endpoint;
- `obt_enabled`;
- command ownership;
- response targets for successful response bodies.

For example:

```yaml
service: praebere
obt_enabled: true

commands:
  models:
    response_targets:
      - originator
      - rogare
```

Nuntius consumes this configuration but does not define it.

## 4. Bootstrap

On startup Nuntius sends Servire a direct:

```text
\obt services
```

Servire returns the active control-plane catalogue.

Healthy startup includes:

```text
services = []
```

provided Servire was reachable and the catalogue request succeeded.

Failure to establish communication with Servire is an unhealthy startup condition.

## 5. Runtime Routing Dictionary

Nuntius should maintain an in-memory routing dictionary.

Conceptually:

```text
replay list
    owner: repetere
    response_targets:
        - originator

models
    owner: praebere
    response_targets:
        - originator
        - rogare
```

Nuntius should not require domain-specific command logic.

## 6. Runtime Topology Updates

When Servire starts/stops a service, it sends a `\obt` topology update through Pontis.

Nuntius updates its routing dictionary.

Nuntius must also be able to discard and rebuild its dictionary by requesting the full catalogue from Servire again.

## 7. Common Service Control Endpoint

Introduce or standardise a common `\obt` endpoint across participating services.

A minimal capability flag is required:

```text
obt_enabled = false
```

Behaviour:

```text
obt_enabled = false
    -> 204 immediately

obt_enabled = true
    -> invoke local command handler
```

An enabled service still returns `204` for a command it does not recognise.

## 8. Response Semantics

Implement and test:

```text
204
    command not handled

200, no body
    command executed successfully and applied

200 + body
    query/request handled successfully; body is result

4xx / 5xx
    recognised command failed

504
    authoritative owner response not received before timeout
```

The distinction between a `200` with and without a body is part of the M0.1 `\obt` contract.

For every solicited command, Nuntius must return one terminal outcome to the originator. A `200` with no body must therefore be propagated back to the originating service or, through Pontis, to the originating external client.

Success must never be inferred from silence.

## 8.1 Timeout Behaviour

Nuntius must apply a configurable timeout while waiting for the authoritative command owner.

On timeout:

```text
Nuntius -> originator: 504 Gateway Timeout
```

The semantic meaning is:

> **The command outcome could not be confirmed within the allowed time.**

Nuntius must not claim that the command definitely failed, because the owner may have applied the operation without returning its response in time.

Once the timeout has been returned, the request is terminal from the originator's perspective.

A late owner response:

- must not be returned later as success for that completed request;
- must be recorded in Nuntius diagnostics;
- may be used for diagnosis but does not alter the original `504`.

Commands that establish prerequisites must treat `504` as failure to establish the prerequisite.

## 9. Pontis Changes

Pontis must:

- recognise `\obt` traffic;
- forward it to Nuntius;
- keep it out of the normal ask/answer route;
- preserve client/session/request correlation;
- accept terminal command outcomes from Nuntius, including `200` with no body;
- deliver those outcomes to the correct originating client.

Pontis must not know the domain owner of individual commands.

## 10. Targeted Command Routing

The initial design should use Servire-defined command ownership to route directly to the appropriate service.

Do not broadcast every command to every active service as the normal path.

`204` remains a defensive/default service response, not the primary discovery mechanism.

## 11. Shared Query Responses

For `200 + body` responses, Nuntius must:

1. return the authoritative result to Pontis for the originating client;
2. inspect the configured response targets;
3. send the same response to configured Lumen consumers where required.

Example:

```text
\obt models
    owner: praebere
    response_targets:
        - originator
        - rogare
```

This avoids broadcasting large provider/model lists to services that do not require them.

## 12. Praebere Integration

Early commands include:

```text
\obt providers
\obt models
\obt model select <model>
```

Praebere owns provider/model semantics.

Likely M0.1 response configuration:

```text
providers
    -> originator
    -> rogare

models
    -> originator
    -> rogare
```

Final consumers should be confirmed during implementation rather than assumed unnecessarily.

## 13. Repetere / Moderari Integration

Repetere must use Nuntius before replay execution to establish Moderari Pass-through for the replay session.

This is a blocking prerequisite. Repetere must receive Moderari's authoritative `200 OK` through Nuntius before beginning any replay model interaction.

`204`, `4xx`, `5xx`, `504`, timeout, or unreachable owner state must abort the replay and leave the run `FAILED / INCOMPLETE`.

The objective is to reproduce the effective system prompt stored in the source Trace.

Required validation:

- perform a fresh normal execution;
- inspect the resulting Trace;
- confirm that the actual effective system prompt is present;
- if absent, update Vestigare before Replay fidelity is considered complete.

The replay Pass-through override must be session-scoped.

## 14. Trace Requirements

Tests must confirm that Nuntius control traffic does not appear as conversational Trace entries.

Tests must separately confirm that execution conditions established through control commands remain represented in Trace.

Examples:

- selected model;
- selected provider;
- effective system prompt.

## 15. Nuntius Diagnostics and Logging

Nuntius must have its own diagnostic logging for the control-command transport.

Do not place detailed Nuntius command lifecycle traffic in the default Servire operational log.

Do not place Nuntius control traffic in Vestigare conversational Trace.

Use the boundary:

> **Servire Ops log = what matters operationally to the Lumen operator.**  
> **Nuntius diagnostics = what happened inside the control-command transport.**  
> **Vestigare Trace = what happened in the model execution.**

### 15.1 Diagnostic Fields

For each solicited command, capture where available:

- timestamp;
- request ID;
- origin/originator;
- session ID where relevant;
- command;
- resolved owner;
- response targets;
- dispatch time;
- terminal response;
- elapsed time;
- timeout state;
- routing/result-delivery outcome;
- late response after timeout;
- routing/configuration errors.

### 15.2 Minimal Diagnostics UI

Implement a lightweight Nuntius diagnostics UI/view showing at least:

- Nuntius health;
- Servire connectivity;
- active route count;
- in-flight command count;
- recent command records.

A command record should expose enough detail to distinguish:

```text
completed
not handled
rejected
owner error
timeout / outcome unconfirmed
late response
routing failure
```

The initial UI is a development/diagnostic surface, not an operator workflow replacement for Servire.

### 15.3 Retention

A bounded diagnostic history is sufficient for M0.1.

Retention should be configurable.

Do not introduce a durable broker or persistent audit subsystem solely for Nuntius diagnostics in M0.1.

### 15.4 Deferred Servire Log Classification

Do not add `log_class` or equivalent metadata to the initial M0.1 Nuntius contract.

The later Servire/Rogare operational-log work will address hiding internal polling/maintenance activity from the default user-facing operational view.

## 16. Testing Requirements

At minimum:

- bootstrap with populated catalogue;
- bootstrap with empty catalogue;
- failed Servire bootstrap;
- route addition/removal;
- full catalogue rebuild;
- `obt_enabled=false` returns `204`;
- enabled service handles owned command;
- unknown command route;
- owner unavailable;
- `200` with no body returned to the originator;
- `200` with body returned to the originator;
- service `4xx/5xx` returned to the originator;
- configurable owner timeout returns `504`;
- timeout is represented as outcome unconfirmed, not confirmed failure;
- late response after timeout is diagnosed but not returned as delayed success;
- Pontis request/session correlation;
- result returned to correct originating client;
- shared query result sent only to configured consumers;
- model/provider list response to Rogare when configured;
- no uncontrolled broadcast of large response bodies;
- Replay Pass-through command delivered to Moderari;
- Replay does not begin until Moderari `200 OK` is returned through Nuntius;
- Replay aborts on `204`, `4xx`, `5xx`, `504`, timeout or unreachable owner;
- failed Replay prerequisite produces `FAILED / INCOMPLETE`;
- Replay Pass-through is session-scoped;
- control traffic excluded from conversational Trace;
- execution conditions remain captured;
- Nuntius diagnostic record created for solicited command lifecycle;
- diagnostic UI exposes health, routes, in-flight count and recent commands;
- timeout and late-response diagnostics are visible;
- Nuntius diagnostic traffic is not copied into Servire Ops log or Vestigare Trace;
- existing Replay/Moderari `\obt` behaviour remains functional after migration.

## 17. Implementation Sequence

1. inspect current Repetere / Moderari / Pontis `\obt` behaviour;
2. define the common response contract;
3. define the Servire control-plane configuration shape;
4. add `obt_enabled` support to service definitions/endpoints;
5. implement Nuntius HTTP skeleton and health;
6. implement direct Servire bootstrap;
7. implement Nuntius in-memory routing dictionary;
8. update Pontis to route `\obt` to Nuntius and preserve request/session correlation;
9. implement targeted command routing;
10. implement terminal response routing to originators, including `200` with no body;
11. implement configurable timeout and `504` outcome;
12. implement configured shared-response distribution;
13. implement Nuntius diagnostic logging and minimal diagnostics UI;
14. migrate existing Repetere/Moderari command behaviour;
15. implement blocking Replay system-prompt Pass-through acknowledgement through Nuntius;
16. implement Praebere provider/model discovery and selection;
17. verify Servire Ops / Nuntius diagnostics / Vestigare Trace separation;
18. complete integration and regression tests.

## 18. Explicitly Out of Scope

The initial Nuntius implementation does not require:

- RabbitMQ;
- Kafka;
- durable queues;
- guaranteed delivery;
- distributed consensus;
- subscriptions;
- authoritative shared-state storage;
- service lifecycle control;
- tool discovery;
- Servire operational-log user/internal classification.

## 19. Development Constraint

Nuntius must remain a routing service.

If it begins to understand model semantics, replay semantics, prompt semantics, service lifecycle policy, or the meaning of returned domain data, responsibility has crossed the intended boundary.
