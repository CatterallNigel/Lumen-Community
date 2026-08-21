# Lumen Integration Contracts Overview

**Status:** Current Supporting\
**Purpose:** Index the architectural contracts at Lumen's service and
external-provider boundaries\
**Last reviewed:** 2026-08-21

------------------------------------------------------------------------

## Purpose

Lumen is deliberately model-, provider- and capability-agnostic. That
independence depends on explicit boundaries between services and on
keeping provider-specific behaviour behind the appropriate Lumen
abstraction.

This document is a **contract index**, not an API reference. It
identifies the responsibility at each important integration boundary,
what crosses that boundary, and what must not leak across it.

For architectural principles, see
`docs/architecture/core/ARCHITECTURE.md`.\
For current topology, ports and service maturity, see
`docs/architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md`
and `LUMEN_SERVICE_MATRIX.md`.

------------------------------------------------------------------------

## Contract Principles

The following rules apply across Lumen integrations:

1.  **Responsibility remains bounded.** An integration must not silently
    transfer one service's responsibility into another.
2.  **Provider-specific behaviour stays behind an abstraction.**
    Model-provider lifecycle belongs behind Praebere; protocol/session
    bridging belongs behind Pontis.
3.  **Evidence remains attributable.** Integration activity that can
    materially affect an execution must remain correlatable with the
    resulting evidence.
4.  **Transport is not responsibility.** HTTP, ACP, broker transport or
    local process communication may change without redefining the
    service contract.
5.  **External capability providers return results or semantics, not
    Lumen orchestration prompts.**
6.  **Moderari owns controlled orchestration at the model boundary.**
7.  **Operational control remains separate from conversational
    execution.**

------------------------------------------------------------------------

## Current Contract Index

  ----------------------------------------------------------------------------------------------------------------
  Boundary           Contract purpose      Primary inputs        Primary outputs             Architectural
                                                                                             constraint
  ------------------ --------------------- --------------------- --------------------------- ---------------------
  Client ↔ Rogare    Human interaction     User requests,        Responses, status,          Rogare is a client,
                     with Lumen            operational actions   user-facing results         not the execution or
                                                                                             assurance engine

  Rogare ↔ Pontis    Submit/correlate      Requests/session      Correlated responses        Pontis owns
                     conversational        context                                           bridge/session
                     activity                                                                behaviour

  Pontis ↔ execution Bridge client traffic Correlated requests   Correlated responses        Bridging must not
  path               into Lumen                                                              reinterpret model
                                                                                             semantics

  Vestigare ↔        Capture execution     Request/response      Trace evidence              Vestigare records; it
  execution path     evidence              activity and relevant                             does not assess
                                           metadata                                          

  Repetere ↔         Controlled            Recorded              Replay result and           Replay is not
  recorded execution reproduction          execution/reference   divergence evidence         repeated-experiment
                                           conditions                                        scheduling

  Fiducia ↔ Repetere Coordinate repeated   Replay target,        Set of controlled replay    Fiducia coordinates
                     controlled runs       schedule/run          executions                  repetition; Repetere
                                           parameters                                        performs each replay

  Aestimare ↔        Behavioural           Trace, Replay,        Assessments                 Aestimare judges
  evidence sources   assessment            repeated-run and                                  evidence; it does not
                                           provenance evidence                               become the
                                                                                             recorder/replayer

  Moderari ↔         Controlled model      Prepared model        Provider request/response   Provider-specific
  model-provider     orchestration         request,                                          lifecycle must remain
  boundary                                 orchestration policy                              outside Moderari

  Praebere ↔ model   Provider              Provider/model        Readiness, availability and External provider
  provider           lifecycle/readiness   lifecycle requests    lifecycle result            remains external to
                     abstraction                                                             Lumen

  Moderari/Pontis ↔  External              Capability invocation Result or semantics         Provider returns
  external           tool/capability use                                                     capability
  capability                                                                                 semantics/results,
  provider                                                                                   not Lumen prompts

  Servire ↔ Lumen    Operational           Start, stop, restart, State, acknowledgement,     Servire is the
  services           lifecycle/control     validation and        logs/status                 control plane, not
                                           operational requests                              the conversational
                                                                                             path

  Audire ↔           Proposed distributed  Operational           Routed                      Audire changes
  Servire/services   operational transport messages/process      messages/acknowledgements   transport, not
                                           output                                            Servire's
                                                                                             responsibility
  ----------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## Model Provider Contract --- Praebere

Praebere is the Lumen-side abstraction between Lumen and external model
providers.

Its contract exists so that the rest of Lumen does not need to
understand provider-specific lifecycle behaviour.

Current responsibilities include:

-   provider status/readiness;
-   model availability;
-   provider start/stop where Lumen owns that lifecycle;
-   model load/unload;
-   enforcing the ownership rule that a provider Lumen did not start
    must not be stopped by Lumen.

The model provider is **not a Lumen service**. Praebere represents the
boundary.

Provider-specific implementation details belong in Praebere
documentation rather than being duplicated here.

------------------------------------------------------------------------

## Client and Session Contract --- Pontis

Pontis provides the bridge between clients, Lumen execution services and
external capability/tool paths.

Its architectural contract includes:

-   preserving request/session correlation;
-   bridging supported protocols;
-   keeping client-specific concerns out of downstream services;
-   providing the path used when Lumen must reach an external capability
    provider.

Pontis must not become the owner of orchestration policy, evidence
assessment or provider lifecycle.

------------------------------------------------------------------------

## Orchestration Contract --- Moderari

Moderari owns controlled orchestration at the model boundary.

Where a request requires Lumen-controlled intervention, Moderari is the
service that applies that intervention. This includes the architectural
rule established for external semantic capability providers:

> The provider returns semantics, not prompts.

An external capability may identify or return meaning, data or a result.
The construction and injection of any Lumen-controlled model instruction
remains Lumen's responsibility.

This prevents external tools from becoming implicitly coupled to Lumen's
prompt architecture.

------------------------------------------------------------------------

## Evidence Contracts

### Vestigare

Vestigare's output is evidence of an execution. Consumers may inspect or
assess that evidence, but Vestigare itself does not decide whether the
behaviour was good, bad, sufficient or trustworthy.

### Repetere

Repetere consumes a recorded execution and reproduces it under
controlled conditions. Its output is Replay/divergence evidence.

A Replay is one execution. Repetere does not own "run this N times"
experiment orchestration.

### Fiducia

Fiducia coordinates repeated Replay activity. It turns repeated
controlled executions into a body of experimental evidence without
absorbing Repetere's execution responsibility.

### Aestimare

Aestimare consumes accumulated evidence and produces behavioural
assessment. It may coordinate multiple specialist internal assessment
engines, but the external service contract remains Aestimare.

------------------------------------------------------------------------

## Operational Contract --- Servire and Audire

Servire owns operational control of the Lumen stack.

Operational commands, dependency validation, state and logs belong to
the control plane and must remain separate from the model request path.

Audire is a proposed transport abstraction for distributed operation. If
introduced, services may publish operational output and receive control
messages through Audire rather than relying on local stdin/stdout/stderr
visibility.

That transport change must not move operational authority from Servire
to Audire.

------------------------------------------------------------------------

## Contract Documentation Rule

This overview records **architectural contracts**.

When an integration becomes implemented and stable enough to require a
concrete API/schema contract, its service documentation should define:

-   endpoint or protocol;
-   request schema;
-   response schema;
-   correlation identifiers;
-   error behaviour;
-   timeout/retry expectations;
-   versioning expectations;
-   provenance/evidence fields where applicable.

This document should then link to that implementation contract rather
than duplicate it.

------------------------------------------------------------------------

## Current Limitation

Lumen does not yet have one uniform formal interface specification for
every service boundary. Several contracts are presently embodied in
individual architecture documents and implementation behaviour.

That is acceptable at the current stage, but implemented interfaces
should progressively be made explicit as they stabilise.
