# Lumen Nuntius — Overview

**Status:** Proposed  
**Service:** Nuntius  
**Role:** Lumen Control-Command Routing and Response Service

## Purpose

Lumen Nuntius is the lightweight internal service responsible for routing Lumen `\obt` control commands and their responses across the currently active Lumen service topology.

`\obt` is the common Lumen control-command language.

Nuntius does not own domain state, execute domain-specific commands, manage service lifecycle, participate in model conversations, or provide durable messaging.

Its responsibility is deliberately narrow:

> **Given a Lumen control command, route it to the service that owns it and return or distribute the resulting response according to the active control-plane configuration supplied by Servire.**

## Why Nuntius Exists

Replay, Moderari, Praebere and other Lumen services need to issue or consume `\obt` commands.

Without a common control plane, those capabilities become point-to-point integrations in which callers must know:

- which service owns a command;
- where that service is running;
- whether a response should return only to the caller;
- whether that response must also be shared with another Lumen service.

Nuntius removes that coupling.

## Servire Defines the Routing Topology

Servire already describes and manages the operational Lumen stack.

Servire therefore also defines the control-plane metadata required by Nuntius.

For each `\obt`-capable service, Servire may describe:

- whether `\obt` handling is enabled;
- the service control endpoint;
- the commands owned by the service;
- which successful command responses return only to the originator;
- which successful response bodies are also required by other services.

Conceptually:

```text
Praebere
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

        model select:
            response_targets:
                - originator
```

Nuntius materialises this configuration into an in-memory routing dictionary.

Nuntius does not decide the policy.

## Command and Response Semantics

The basic service response contract is:

```text
204 No Content
    This service does not handle the command.

200 OK, no body
    The command was executed successfully.

200 OK, with body
    The command is a request/query and the body is the result.

4xx / 5xx
    The command was recognised but failed.
```

For a `200` response with a body, Nuntius returns the result to Pontis. Pontis owns external session/client routing and sends the result to the originating client.

If Servire configuration declares that the response is also required by another Lumen service, Nuntius sends the same authoritative response to those configured consumers.

## Example — Replay List

```text
External Client
      |
      | \obt replay list
      v
    Pontis
      |
      v
    Nuntius
      |
      v
   Repetere
      |
      | 200 + replay list
      v
    Nuntius
      |
      v
    Pontis
      |
      v
External Client
```

The result belongs only to the requesting session unless Servire explicitly configures another consumer.

## Example — Model List

A model list is initiated by one client but may also be required by Rogare.

```text
External Client
      |
      | \obt models
      v
    Pontis
      |
      v
    Nuntius
      |
      v
   Praebere
      |
      | 200 + model list
      v
    Nuntius
      |
      +----> Pontis -> originating client
      |
      +----> Rogare
```

Praebere remains authoritative for the model list.

Nuntius transports the result but does not own or cache model-domain state as an authority.

## Pontis Responsibility

Pontis remains the external session bridge.

For `\obt` traffic:

```text
Client -> Pontis -> Nuntius
```

For returned query results:

```text
Service -> Nuntius -> Pontis -> originating session/client
```

The boundary is:

> **Pontis owns client/session correlation. Nuntius owns command/service routing and response distribution.**

## Bootstrap

Servire is authoritative for the active service and command topology.

On startup Nuntius contacts Servire directly:

```text
\obt services
```

Servire may return a populated control-plane catalogue or an empty catalogue.

An empty catalogue is a healthy Nuntius startup state provided communication with Servire succeeds.

As Servire starts or stops services, it sends control-plane catalogue updates through Pontis to Nuntius.

## Trace Boundary

Nuntius is part of the control plane, not the normal ask/answer path.

Internal `\obt` traffic must not appear as conversational turns in Vestigare Trace records.

Where a control command changes a model execution condition, the resulting execution condition remains part of the Trace.

For example, the command:

```text
\obt model select qwen2.5-coder:14b-32k
```

does not itself become a conversational Trace entry.

The subsequent execution still records the model/provider actually used.

The same principle applies to system prompts: the control command used to establish the policy is not conversational evidence, but the actual system prompt supplied to the model is.

## Deliberate Non-Responsibilities

Nuntius is not:

- a replacement for Servire;
- a service lifecycle manager;
- a durable message broker;
- an authoritative state store;
- a model-provider abstraction;
- a tool catalogue;
- a conversational proxy;
- a Trace transport;
- an operational-log classifier.

Operational-log classification such as distinguishing user activity from internal polling is a separate Servire/Rogare concern and is not part of the initial Nuntius implementation.

## Core Principle

> **Servire defines the active control-plane topology. Nuntius routes commands and responses according to that topology. Domain services execute their own commands, and Pontis returns client-facing responses to the correct session.**
