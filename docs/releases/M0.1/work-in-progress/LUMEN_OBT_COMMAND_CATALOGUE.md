# Lumen `/obt` Command Catalogue

**Status:** Living canonical register  
**Initial catalogue date:** 2026-08-27  
**Scope:** User-facing and internal Lumen control-plane commands

## Document Revision History

| Date | By | Version | Description |
| --- | --- | --- | --- |
| 2026-08-27 | Nigel Catterall / OpenAI | 0.1 | Initial catalogue assembled from the Nuntius architecture, M0.1 roadmap, and recorded Repetere command decisions |

## 1. Purpose

This document is the canonical dictionary of Lumen `/obt` commands.

It records:

- the exact command syntax;
- the service that owns and executes the command;
- whether the command is user-facing or internal;
- its arguments and response;
- its implementation state;
- unresolved syntax or behaviour that must be confirmed from code.

Every new `/obt` command must be added here when it is designed, not after it is implemented.

## 2. Prefix Reconciliation Required

The current architectural documents render the prefix as `\obt`, while the current catalogue request calls it `/obt`.

This catalogue uses `/obt` as its display form, but the accepted wire syntax must be verified against the current Pontis, Repetere, and Moderari implementations before Version 1.0. Until then, `\obt` must be treated as a possible legacy or actual wire prefix rather than silently discarded.

## 3. Status Definitions

| Status | Meaning |
| --- | --- |
| Implemented | Previously recorded as supported by the existing service |
| Proposed | Required or described by an approved architecture/roadmap but not confirmed in code |
| Internal | Intended for service-to-service control-plane use rather than normal user entry |
| Needs code verification | Existing behaviour or precise syntax must be established by inspecting the current repositories |
| Reserved | Name is retained, but no executable contract has yet been defined |

## 4. Command Grammar

```text
/obt <domain> [action] [arguments...]
```

Known shorter query forms also exist:

```text
/obt services
/obt providers
/obt models
```

The external command must not expose the owning service's endpoint. Pontis recognises the command, Nuntius routes it using Servire's authoritative catalogue, and the domain service executes it.

## 5. Command Index

| Command | Owner | Audience | Type | Status |
| --- | --- | --- | --- | --- |
| `/obt services` | Servire | Nuntius / operator | Query | Proposed; existing form needs verification |
| `/obt service available ...` | Servire → Nuntius | Internal | Topology notification | Proposed, Internal |
| `/obt service unavailable ...` | Servire → Nuntius | Internal | Topology notification | Proposed, Internal |
| `/obt providers` | Praebere | Client / Rogare | Query | Proposed |
| `/obt models` | Praebere | Client / Rogare | Query | Proposed |
| `/obt model select <model>` | Praebere | Client / Rogare | State change | Proposed |
| `/obt replay` | Repetere | Client / Rogare | Help | Implemented; needs code verification |
| `/obt replay help` | Repetere | Client / Rogare | Help | Implemented; needs code verification |
| `/obt replay list` | Repetere | Client / Rogare | Query | Implemented; needs code verification |
| `/obt replay start <replay-id>` | Repetere | Client / Rogare | State change | Implemented; needs code verification |
| `/obt replay status` | Repetere | Client / Rogare | Query | Implemented; needs code verification |
| `/obt replay stop` | Repetere | Client / Rogare | State change | Implemented; needs code verification |
| `/obt replay unstage <name-or-id>` | Repetere | Client / Rogare | State change | Reserved; exact syntax unresolved |
| `/obt replay delete <name-or-id>` | Repetere | Client / Rogare | State change | Reserved; exact syntax unresolved |
| Moderari session Pass-through command | Moderari | Repetere | State change | Proposed, Internal; syntax unresolved |

## 6. Servire Commands

### 6.1 `/obt services`

| Field | Definition |
| --- | --- |
| Owner | Servire |
| Purpose | Returns the authoritative active Lumen control-plane catalogue |
| Primary caller | Nuntius during bootstrap or catalogue rebuild |
| Expected success | `200 OK` with catalogue body; an empty catalogue is valid and healthy |
| Failure significance | Nuntius cannot establish authoritative routing during bootstrap |
| Status | Proposed extension; inspect current implementation |

The response must contain enough information to determine:

- configured and running services;
- service endpoints;
- whether `obt_enabled` is true;
- command ownership;
- successful response targets.

### 6.2 `/obt service available ...`

Internal runtime notification that a service has become available. The payload must contain enough information for Nuntius to add or update its routes. Exact syntax and payload schema remain unresolved.

### 6.3 `/obt service unavailable ...`

Internal runtime notification that a service is no longer available. The payload must identify the service and allow Nuntius to remove its active command routes. Exact syntax and payload schema remain unresolved.

## 7. Praebere Commands

### 7.1 `/obt providers`

| Field | Definition |
| --- | --- |
| Owner | Praebere |
| Purpose | Returns available model providers |
| Expected success | `200 OK` with authoritative provider list |
| Expected response targets | Originator and Rogare where configured by Servire |
| Status | Proposed for M0.1 |

### 7.2 `/obt models`

| Field | Definition |
| --- | --- |
| Owner | Praebere |
| Purpose | Returns available models for the applicable provider context |
| Expected success | `200 OK` with authoritative model list |
| Expected response targets | Originator and Rogare where configured by Servire |
| Status | Proposed for M0.1 |

Open question: determine whether provider is implicit in shared state or becomes an explicit command argument.

### 7.3 `/obt model select <model>`

| Field | Definition |
| --- | --- |
| Owner | Praebere |
| Purpose | Selects the model used for subsequent execution |
| Argument | `<model>` — exact provider-qualified identifier format unresolved |
| Expected success | `200 OK` with no body |
| Evidence requirement | Subsequent Trace evidence must record the actual selected provider/model execution state |
| Status | Proposed for M0.1 |

The architecture's configuration example uses the internal key `model_select`. That key must not be mistaken for the public command syntax.

## 8. Repetere Commands

### 8.1 `/obt replay`

Bare replay command. Previously defined to behave as help rather than start a replay.

### 8.2 `/obt replay help`

Returns Replay command help and supported syntax.

### 8.3 `/obt replay list`

Returns prepared Replay sessions in the `ready` state.

Expected success: `200 OK` with an authoritative list body.

### 8.4 `/obt replay start <replay-id>`

Starts the identified prepared replay.

Known constraints:

- an unnamed session cannot be prepared;
- duplicate prepared-session names are not permitted;
- M0.1 requires each Replay execution to receive a new isolated Replay session;
- Replay must not inherit context or temporary state from a preceding run.

### 8.5 `/obt replay status`

Returns the status of the current or applicable replay. The exact response schema and whether an optional replay identifier is supported require code verification.

### 8.6 `/obt replay stop`

Stops the active replay. The exact scope when several sessions exist requires code verification.

### 8.7 Reserved Repetere lifecycle commands

Earlier design decisions require operations to unstage and delete prepared sessions, but their implemented syntax has not been established.

Reserved candidate forms:

```text
/obt replay unstage <name-or-id>
/obt replay delete <name-or-id>
```

These are not to be treated as implemented until verified.

## 9. Moderari Commands

### 9.1 Session-scoped Pass-through override

Before replay begins, Repetere must instruct Moderari through Nuntius to apply Pass-through to the newly created Replay session.

Required semantics:

- session-scoped, never global;
- applies before any Replay model interaction;
- requires authoritative `200 OK` acknowledgement;
- `204`, `4xx`, `5xx`, or timeout aborts the run as `FAILED / INCOMPLETE`;
- the effective system prompt from the source Trace is then replayed exactly once.

The architecture illustrates this semantically as `Moderari: Pass-through`, but no final public or wire syntax is defined. A candidate command must not be invented here. The current Moderari injection path must be inspected first.

## 10. Common Response Contract

| Response | Meaning |
| --- | --- |
| `200 OK`, no body | Command executed successfully |
| `200 OK` with body | Query completed; body is the authoritative result |
| `204 No Content` | Target service did not handle the command |
| `4xx` / `5xx` | Owning service recognised the command but failed to execute it |
| `504 Gateway Timeout` | Outcome could not be confirmed before timeout |

Every solicited command must produce one terminal response to its originator. Silence is never success.

A late owner response after `504` is diagnostic evidence only. It must not replace the terminal timeout already returned to the originator.

## 11. Routing and Trace Rules

- Servire is authoritative for active command ownership and response consumers.
- Pontis recognises external `/obt` traffic and forwards it to Nuntius.
- Nuntius routes to the configured owner and correlates the result.
- Nuntius does not interpret command-domain data.
- A service with `obt_enabled = false` returns `204` immediately.
- Unknown or irrelevant commands return `204`; recognised failures return an error.
- Control commands do not become user, assistant, system, or tool turns in Vestigare Trace.
- The execution conditions produced by commands remain observable in Trace where required for reproduction.
- Nuntius diagnostics remain separate from both the Servire Operations Log and Vestigare conversational Trace.

## 12. Commands Not Yet Catalogued

No executable `/obt` contract has yet been established here for:

- Fiducia schedules and repeated Experiment runs;
- Vestigare trace control;
- Rogare UI control;
- Aestimare or Periti assessment operations;
- provider selection distinct from model selection;
- inference configuration;
- saved Moderari prompt CRUD;
- service lifecycle start, stop, or restart through Servire.

These are gaps to investigate, not implicit commands.

## 13. Required Code-Audit Checklist

Before promoting this catalogue to Version 1.0, search every active Lumen repository for:

```text
/obt
\obt
obt_
"obt"
replay help
replay list
model select
pass-through
```

For every match:

1. identify the accepting endpoint or parser;
2. record the exact wire syntax and aliases;
3. record arguments, defaults, validation, and response schema;
4. record the executing service and downstream side effects;
5. add tests that prove the catalogue entry;
6. remove or explicitly deprecate conflicting syntax;
7. update this document in the same change.

## 14. Command Registration Template

Copy this block when adding a command:

```markdown
### `/obt <domain> <action> [arguments]`

| Field | Definition |
| --- | --- |
| Owner | `<service>` |
| Audience | `External / Rogare / Internal service` |
| Purpose | `<single responsibility>` |
| Arguments | `<names, types, valid values, defaults>` |
| Expected success | `<status and response schema>` |
| Errors | `<status, conditions, retry semantics>` |
| Response targets | `<originator and configured consumers>` |
| Trace effect | `<none, or resulting execution evidence>` |
| Status | `<Implemented / Proposed / Internal / Reserved>` |
| Tests | `<repository and test identifiers>` |
```

## 15. Canonical Maintenance Rule

> A `/obt` command is not complete until its syntax, owner, arguments, response contract, trace effect, implementation status, and tests are recorded in this catalogue.

