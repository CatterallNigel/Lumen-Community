# Lumen Moderari --- IP-01.1 SIE Foundation Architecture Decisions

**Date:** 2026-10-07\
**Status:** Agreed foundation for implementation\
**Scope:** Moderari Standalone Interaction Engine (SIE), CIM, Adapters,
Filters, execution ownership and evidence boundaries

## 1. Purpose

This document records the architecture decisions agreed before
implementation of IP-01.1. Exact Python interfaces, generated-model
shapes, discovery paths and similar implementation details remain to be
proven by implementation.

The objective is to keep Moderari Core, SIE, Adapters and Filters
sharply separated while supporting multiple client and provider/model
protocols without making Filters protocol-aware.

## 2. Conversation and Interaction Ownership

The **client owns the authoritative conversation context**. Pi-like
clients supply their complete applicable context on each Ask. For a
deliberately thin client such as Rogare, Moderari may fulfil the
client-side transcript responsibility.

-   **Client:** authoritative conversation history/context.
-   **Moderari Core:** provider transport/execution and Core/SIE
    boundary.
-   **SIE:** transient flow control for the current interaction.
-   **Adapter:** mapping between its supported external representation
    and CIM.
-   **Filter:** operates on CIM; does not own the conversation.
-   **Provider/model:** receives the effective interaction for the
    current execution.

Filter transformations do not create a second persistent conversation
history.

## 3. Canonical Interaction Model

### 3.1 Normative Definition

> **CIM is a versioned, schema-defined canonical interaction data model.
> The schema is authoritative; language-specific representations may be
> generated from it. Adapters map external protocol representations to
> and from CIM. Adding support for a protocol requires a new Adapter,
> but requires a CIM change only when that protocol introduces
> interaction semantics that the current CIM cannot represent and Lumen
> chooses to preserve.**

CIM is not an Adapter, service, transformation process or routing
mechanism. It is the canonical data definition through which supported
external interaction representations interoperate.

### 3.2 Mapping Model

``` text
Protocol A ─┐
Protocol B ─┼── many → one ──→ CIM
Protocol C ─┘

CIM ── one → many ──┬──→ Protocol A
                    ├──→ Protocol B
                    └──→ Protocol C
```

There is no direct pairwise protocol mapping. Interoperability emerges
from independent mappings to and from CIM.

CIM represents interaction semantics rather than wrapping one protocol's
syntax. A new protocol does not automatically require a CIM change. CIM
evolves only when Lumen chooses to preserve meaningful interaction
semantics that the current CIM cannot represent.

### 3.3 CIM as the Filter Boundary

CIM is the mandatory Filter boundary:

``` text
CIM → Filter → CIM
```

This eliminates the previously envisaged Adapter/Filter loopback as a
separate mechanism. Every Filter accepts and returns CIM, so there is
one deterministic execution path:

``` text
Moderari Core
      ↓
     SIE
      ↓
 Adapter A
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
 Adapter B
      ↓
     SIE
      ↓
Moderari Core
```

### 3.4 CIM Version Scope

IP-01 supports **one active CIM version**. The Session Execution Plan
records the exact CIM version used. Adapters and Filters declare the CIM
version they support.

For the present architecture:

``` text
CIM: exactly one active version
Adapter identity: exactly one installed version
Filter identity: exactly one installed version
```

Versions are provenance and compatibility facts, not runtime selection
dimensions. Multiple CIM versions, version negotiation and multiple
installed versions of one Adapter or Filter identity are deferred.

## 4. Adapter Responsibilities

### 4.1 Base Protocol Adapter

A Base Protocol Adapter maps bidirectionally between one supported
external protocol representation and CIM:

``` text
IN:  external protocol representation → CIM
OUT: CIM → external protocol representation
```

IN and OUT are relative to CIM, not client/model or request/response.

An Adapter knows its supported external representation and the CIM
contract. It does not know the opposing protocol, whether it is
logically Adapter A or B, whether material originated from a client or
model, what other Adapters exist, or the wider route.

> **An Adapter knows its supported external representation and the CIM
> contract. It knows nothing about the opposing protocol or the wider
> interaction route.**

Cross-protocol interoperability is:

``` text
Protocol A → Base Adapter A → CIM → Base Adapter B → Protocol B
```

### 4.2 Specialised Adapter

A Specialised Adapter is a **same-protocol compatibility mechanism** for
provider/model-specific deviations.

It may only be selected when:

``` text
Client protocol == Specialised Adapter base protocol == Provider/Model protocol
```

A Specialised Adapter: - specialises one existing Base Protocol
Adapter; - inherits that Base Protocol; - does not define a new
protocol; - does not extend CIM; - is eligible only when the same
protocol exists on both sides and its provider/model selector matches; -
never participates in cross-protocol translation.

Architecturally, one resolved Specialised Adapter occupies, in a virtual
sense, both Adapter A and Adapter B roles:

``` text
Same Protocol Client
        ↓
       SIE
        ↓
 Specialised Adapter
   virtual A ↔ CIM ↔ virtual B
        ↓
       SIE
        ↓
Same Protocol Provider/Model
```

> **A Specialised Adapter is one same-protocol effective Adapter
> occupying the architectural roles that Adapter A and Adapter B would
> otherwise occupy.**

Cross-protocol interactions always use Base Protocol Adapters around
CIM. A Specialised Adapter is not inserted on one side of a
cross-protocol pair.

## 5. Filter Contract and Representation

All Filters consume CIM and produce CIM. There are no protocol-specific
Filters in the SIE architecture.

> **Filters operate on CIM. External protocol representations are
> Adapter concerns.**

Thus:

``` text
CIM → Filter → CIM → Filter → CIM
```

Filter ordering never changes representation, and SIE requires no
representation-routing logic between Filters.

## 6. External and Plugin Filters

A Filter need not be packaged inside a Lumen image. A separately
packaged or mounted Filter may implement the same CIM Filter contract
and be discovered like a Lumen-supplied Filter.

From SIE's perspective there is simply a **discovered Filter**
satisfying the contract:

``` text
Lumen / SIE → CIM → External Filter → CIM → Lumen / SIE
```

This provides plugin capability without requiring a separate plugin
architecture. The Filter manifest and executable Filter contract form
the component boundary; packaging/deployment are separate concerns.

Lumen does not need to publish CIM now. Future options include the full
schema, a stable Filter-facing subset, generated bindings, or a Filter
SDK.

## 7. Filter Behaviour

A transformational Filter changes the current effective interaction:

``` text
CIM 1 → Filter → CIM 2
```

A private-model/observational Filter may require a model interaction:

``` text
Current CIM
    ↓
  Filter
    ↓
 Filter Ask
    ↓
   SIE
    ↓
 Moderari Core
    ↓
 Provider / Model
    ↓
 Moderari Core
    ↓
   SIE
    ↓
 Filter Response
    ↓
  Filter
    ↓
 Filter Result
```

A Filter invocation may have zero, one or multiple private Ask/Response
cycles. SIE correlates them through `filter_execution_id`. An
observational Filter may return the continuing CIM unchanged while
producing evidence/results.

## 8. No Persistent Filter-Created Conversation Branch

> **Filter transformations apply to the current execution. They do not
> establish a persistent alternate conversation-history branch.**

The next normal Ask begins again from the authoritative client context.

## 9. System Prompt Filter

System Prompt is the initial transformational Filter. It ensures the
configured Lumen System Prompt occupies the System Prompt position in
the effective CIM interaction.

Initial policy: - insert the configured Lumen System Prompt at the
beginning of applicable context; or - replace the existing System Prompt
at the beginning of applicable context.

Sophisticated multiple-System-Prompt semantics are out of scope.

``` text
CIM
 ↓
System Prompt Filter
 ├── Insert configured Lumen System Prompt
 └── OR replace existing System Prompt
 ↓
CIM
```

## 10. Cognition Filter

Cognition is observational, not a context-rewriting or compaction
mechanism.

> **What was the model's state of understanding at this point?**

Cognition may construct a private instruction/Ask, route it through
SIE/Core to the selected provider/model, consume the private response,
produce Cognition evidence, and leave the continuing CIM unchanged.

Its evidence can later be used by Aestimare to assess matters such as
drift from user intent.

## 11. Assistant Prompt

A general-purpose Assistant Prompt is **not an independent Filter in
IP-01**.

Where Cognition or another Filter requires a one-off assistant-like
instruction in a private model interaction, that instruction belongs to
that Filter implementation and does not enter the normal client/model
conversation.

A standalone Assistant Prompt plugin may be designed later if a real
requirement emerges.

> **IP-01 provides System Prompt as the initial transformational Lumen
> Filter and Cognition as the initial private-model/observational
> Filter. Assistant Prompt remains an internal implementation mechanism
> where required by a Filter and is not currently a standalone Filter.**

## 12. Filter Runtime State

If a Filter later requires session-scoped operational state, that state
is distinct from conversational Context:

1.  conversation context --- client-owned, or reconstructed by Moderari
    for a thin client;
2.  Filter configuration --- what the Filter is configured to do;
3.  Filter runtime state --- session-scoped operational state owned by
    the Filter implementation.

The resolved Filter instance in the SIE Execution Instance is a natural
future lifetime boundary for such state. IP-01 does not need to invent
stateful behaviour merely to demonstrate it.

## 13. EMD and Filter Applicability

Praebere supplies the current Effective Model Definition (EMD). A Filter
manifest declares only the EMD keys it requires.

SIE determines whether required facts are available. The Filter owns
their meaning and thresholds. SIE does not create a characteristic
framework or interpret model-capability semantics.

A third-party Filter should be able to introduce a required EMD
characteristic, have it supplied in the provider/model definition, and
be discovered/evaluated without changing SIE code.

## 14. Session Execution Plan and Execution Instance

The immutable Session Execution Plan records what was resolved,
including as applicable:

-   client protocol/representation;
-   selected provider;
-   selected model;
-   provider/model protocol;
-   **CIM version**;
-   resolved Adapter identities and versions;
-   resolved Filter identities and versions;
-   relevant immutable execution configuration.

The session-scoped Execution Instance is the minimal hot runtime
machinery SIE needs to follow that Plan. It must not duplicate
provider/model knowledge.

> **SIE remembers what this session has resolved; it does not remember
> what this model is capable of.**

### 14.1 Replay and CIM Version

CIM version is part of the reproducibility contract.

> **Replay against the original setup requires the exact CIM version
> recorded in the original Session Execution Plan. A different available
> CIM version constitutes a different execution environment and
> therefore requires a newly resolved Session Execution Plan.**

This preserves the distinction between Original Setup replay and a New
Setup experiment. Future CIM compatibility/negotiation rules are outside
IP-01.

## 15. Evidence and Persistence Boundaries

### 15.1 Vestigare

Vestigare records all normal client ↔ model traffic in the **client
protocol**.

### 15.2 Moderari

Moderari persists Filter activity/results in the **provider + model
protocol**. There is no requirement to translate those records into the
client protocol merely for persistence.

### 15.3 Interpretation

Vestigare contains the Session Execution Plan. The Plan provides the
protocol and component-version context required to interpret both
evidence streams.

> **Vestigare preserves client/model traffic in the client protocol.
> Moderari preserves Filter activity in the provider/model protocol. The
> Session Execution Plan provides the authoritative protocol and version
> context required to interpret both.**

Cross-representation correlation and assessment belong to Aestimare or
other evidence consumers rather than the execution path.

## 16. SIE Responsibility Boundary

SIE is Moderari Core's single interaction-processing **flow-control**
boundary.

SIE does not perform protocol transformation. Adapters perform all
external representation ↔ CIM mapping. Filters transform or observe CIM.
SIE directs traffic between the resolved components according to the
Execution Instance derived from the Session Execution Plan.

``` text
Moderari Core
      ↓
     SIE
      ↓
 Adapter A
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
 Adapter B
      ↓
     SIE
      ↓
Moderari Core
```

SIE may discover component definitions, accept EMD for
applicability/resolution, resolve the execution path during
materialisation, invoke resolved components, direct CIM between them,
orchestrate private Filter model interactions through Core, correlate
Filter Ask/Response cycles, use the Execution Instance, and dispose
session runtime machinery.

SIE must not perform protocol ↔ CIM conversion itself, interpret/alter
protocol or CIM semantics, implement Filter transformations, own
persistent conversation history, own provider transport, own session
identity/lifecycle, build EMD, own EMD provenance, hardcode concrete
Adapter/Filter semantics, persist directly to Vestigare/MongoDB,
implement protocol-specific Filter routing, or create persistent
alternate conversation branches.

> **SIE is flow control. Adapters map representations. Filters transform
> or observe CIM. SIE itself transforms nothing.**

> **SIE knows the structure and contracts of interaction components. It
> does not know the semantics of their implementations.**

## 17. Component Version Rule

For IP-01, component discovery resolves one installed implementation for
each component identity.

``` text
component ID → one installed implementation → one version
```

Manifest versions are provenance/replay facts, not runtime selection
dimensions.

Multi-version Adapter/Filter coexistence must be revisited together with
any future support for multiple CIM versions.

## 18. Foundation Extensibility Tests

The foundation should prove:

> Given discovered/self-describing Adapter and Filter implementations,
> an authoritative current EMD, selected session configuration and a raw
> first client request, can SIE determine whether the configuration is
> usable, resolve the required Adapter/Filter flow, materialise a
> minimal hot session-scoped Execution Instance, and thereafter direct
> interactions without re-reading model knowledge or knowing concrete
> Adapter, Filter or CIM semantics?

> Can Moderari Core treat SIE as the single interaction-processing
> enter/exit point, including private Filter model interactions, without
> containing protocol-, model-, CIM- or Filter-specific transformation
> logic?

> Can a separately packaged Filter implementing the current CIM Filter
> contract be discovered and executed without SIE knowing whether it is
> Lumen-supplied or externally supplied?

> Can a new protocol be supported by adding a new Adapter mapping that
> protocol to/from current CIM without modifying SIE or existing
> Adapters, provided no new interaction semantic that Lumen chooses to
> preserve requires CIM evolution?

## 19. Decisions Explicitly Deferred

Deferred until implementation evidence or real demand exists:

-   exact Python Adapter interface;
-   exact Python Filter interface;
-   exact generated CIM Python representation;
-   exact Filter result/envelope types;
-   exact common manifest field names;
-   exact discovery/install convention;
-   public CIM publication;
-   full CIM vs public Filter-facing subset/SDK;
-   general-purpose Assistant Prompt Filter;
-   Assistant Prompt scheduling/state semantics;
-   sophisticated multiple-System-Prompt semantics;
-   detailed stateful Filter lifecycle;
-   multiple active CIM versions;
-   CIM version negotiation/compatibility;
-   multiple installed versions of one Adapter identity;
-   multiple installed versions of one Filter identity;
-   runtime component-version selection;
-   detailed Specialised Adapter internals beyond the established
    same-protocol topology/invariants.

## 20. Implementation Starting Point

IP-01.1 can now move into foundation contracts.

``` text
External Interaction
        ↓
       SIE
        ↓
     Adapter
        ↓
       CIM
        ↓
       SIE
        ↓
      Filter
        ↓
       CIM
        ↓
       SIE
        ↓
     Adapter
        ↓
       SIE
        ↓
External Interaction
```

System Prompt proves a simple CIM transformation. Cognition proves the
private Filter Ask/Response lifecycle.

The first CIM implementation establishes the authoritative versioned
schema and the language-specific representation used by Adapters and
Filters. Adding another protocol should require only another Adapter
unless it demonstrates a meaningful interaction semantic that the
current CIM cannot represent and Lumen chooses to preserve.

Implementation details should be added only when demonstrated by
concrete paths rather than speculative future requirements.
