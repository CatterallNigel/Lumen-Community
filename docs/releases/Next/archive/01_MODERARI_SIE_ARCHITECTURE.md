# Lumen Moderari Standalone Interaction Engine (SIE) Architecture

**Status:** Current authoritative architecture  
**Date:** 02026-10-10  
**Scope:** Moderari Standalone Interaction Engine foundation through IP-01.1 Milestone 5E

## 1. Purpose

The Standalone Interaction Engine (SIE) is the interaction-processing boundary inside Moderari. It provides deterministic flow control across Adapters, Specialised Adapters, Filters and the Canonical Interaction Model (CIM).

The governing rule is:

> **SIE is flow control. Adapters map representations. Filters transform or observe CIM. SIE itself transforms nothing.**

SIE knows the structure and contracts of interaction components. It does not know the implementation semantics of those components.

## 2. Responsibility Boundary

Moderari Core remains responsible for service-level orchestration, provider transport and execution. SIE is responsible for selecting and invoking the already-resolved interaction-processing components for a session.

Normal interaction flow is:

```text
Moderari Core
      ↓
     SIE
      ↓
 Effective Adapter IN
      ↓
     CIM
      ↓
     SIE
      ↓
  Filter(s)
      ↓
     CIM
      ↓
     SIE
      ↓
 Effective Adapter OUT
      ↓
     SIE
      ↓
Moderari Core
```

The effective Adapter at either boundary may be a Base Adapter or a resolved Specialised Adapter (SpAd).

## 3. Conversation and Context Ownership

The client remains authoritative for the conversation transcript in the current architecture. Pi, for example, sends the full transcript with each Ask. Rogare is a thin client and Moderari may reconstruct the interaction required for Rogare.

SIE does not own a persistent conversation context. Filters do not own conversation context. There is no generic persistent `Context` object in the SIE foundation and no Filter-created persistent conversation branch.

## 4. Session Configuration and Execution

A Pontis session exists before model selection. Praebere owns provider/model knowledge and the session-to-model reservation. Rogare coordinates model selection and Filter Profile selection.

Before interaction execution, Moderari establishes an immutable **Session Execution Plan**. It contains the exact execution facts required to reproduce and interpret the session, including:

- client interface;
- selected provider and model;
- provider/model interface;
- active CIM version;
- effective Adapter identities and versions;
- selected Filter identities and versions;
- immutable Filter configuration relevant to the session.

SIE creates or uses a session-scoped **Execution Instance** from that Plan. Runtime state is execution state, not model knowledge.

> **SIE remembers what this session has resolved; it does not remember what this model is capable of.**

## 5. Effective Model Definition

Praebere supplies the Effective Model Definition (EMD). Its schema defines the structure while its characteristic vocabulary remains extensible.

Filters may declare required EMD characteristic keys. SIE checks that required keys exist; the Filter owns interpretation of their values. SIE does not maintain a persistent compatibility catalogue.

## 6. Components Known by SIE

SIE recognises component categories and contracts:

- Base Adapters;
- Specialised Adapters;
- Filters;
- the active CIM schema/version.

SIE does not contain protocol-specific translation logic or Filter-specific transformation logic.

### 6.1 Adapter selection

Adapter resolution is based on interface/provider/model identity. Direction does not participate in Adapter identity.

Resolution precedence is:

```text
interface + provider + exact model
              ↓
interface + provider + model family
              ↓
interface Base Adapter
```

The result is one effective Adapter. SIE then invokes its `IN` or `OUT` operation according to the component's position in the interaction path.

### 6.2 Filter selection

Filters are explicitly selected through session configuration / Filter Profiles. They are never automatically selected to compensate for an Adapter incompatibility.

A Filter consumes CIM and returns CIM. It may intentionally transform semantics or may observe CIM and perform private model interaction according to its declared execution mode.

## 7. Interaction Direction

The architectural model is bidirectional:

```text
Client → Adapter IN → CIM → Filters → CIM → Adapter OUT → Provider

Provider → Adapter IN → CIM → Filters (where applicable) → CIM
         → Adapter OUT → Client
```

The production foundation currently proves the request-side CIM path and contains a real Qwen provider-response compatibility normaliser. The complete provider-response CIM execution path remains future SIE integration work; the Qwen outlier does not justify prematurely expanding CIM or the Base Adapter mapping model.

## 8. Configuration Lifecycle

Configuration-time discovery determines which components are available. Session configuration associates the selected model and Filter Profile with the session. First-Ask materialisation resolves exact component identities/versions and creates immutable execution state.

Subsequent Asks use that immutable state. Session end clears the execution instance and triggers the appropriate service cleanup.

## 9. Error Philosophy

SIE must not guess. Ambiguous Adapter resolution, missing required components, incompatible versions, invalid CIM, invalid external representations, or impossible destination materialisation are explicit failures.

A destination that can produce a valid representation while objectively omitting an unrepresentable CIM semantic is not necessarily a failure; Adapter OUT may instead return `PASS_WITH_SEMANTIC_LOSS`. The Adapter architecture defines that distinction.

## 10. Evidence and Persistence

Vestigare stores the normal client↔model interaction in the client-facing representation together with execution metadata required to interpret it.

Moderari owns Filter transformation/activity evidence and other SIE execution evidence. Filters do not write directly to Vestigare merely because they transform CIM.

Exact component identities and versions are provenance. Replay uses recorded execution facts rather than depending on a mutable Filter Profile definition.

## 11. Version Rule

For the current foundation:

- one active CIM version;
- one installed version per Adapter identity;
- one installed version per Filter identity;
- no runtime version negotiation;
- exact versions are recorded for provenance and replay.

Multi-version compatibility is deferred.

## 12. Implemented Foundation

Through Milestone 5E the foundation contains:

- CIM 1.0 candidate and validation;
- OpenAI Chat Completions Base Adapter;
- Anthropic Messages Base Adapter;
- Adapter registry/resolution;
- SpAd inheritance and deterministic resolution;
- real Qwen Tool SpAd;
- BaseFilter production contract;
- real System Prompt Filter;
- conformance harness retained as architectural assurance authority.

SIE component discovery/execution integration is the next architectural implementation area.

## 13. Architectural Invariants

1. SIE performs flow control, not semantic transformation.
2. Adapters change representation while preserving semantics wherever the destination permits.
3. Filters are the explicit mechanism for intentional semantic transformation.
4. All Filters operate on CIM.
5. Adapter resolution is independent of IN/OUT direction.
6. A resolved SpAd is the single effective Adapter at that boundary.
7. SIE does not infer provider/model behaviour.
8. Session execution state is immutable after materialisation.
9. Component boundaries are schema-constrained.
10. No component may silently invent semantics.

## 14. Deferred Work

The following remain deliberately outside the current foundation:

- full provider-response CIM execution path;
- SIE discovery implementation;
- dynamic third-party code loading policy;
- multi-version CIM/Adapter/Filter negotiation;
- broader EMD characteristic semantics;
- private-model observational Filter implementation;
- full integration with Praebere, Pontis, Rogare, Vestigare and Repetere.
