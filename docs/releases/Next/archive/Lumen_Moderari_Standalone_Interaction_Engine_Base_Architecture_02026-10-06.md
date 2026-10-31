# Lumen Moderari Standalone Interaction Engine (SIE)

## Base Architecture and Framework Definition

**Status:** Architecture baseline for `research-foundation-next`\
**Date:** 02026-10-06\
**Workstream:** IP-01 --- Moderari Standalone Interaction Engine

------------------------------------------------------------------------

## 1. Purpose

The Standalone Interaction Engine (SIE) is the interaction-processing
boundary within Moderari.

Moderari Core passes interactions into SIE. SIE determines what, if
anything, must happen to those interactions before they cross the next
external boundary. SIE may perform no transformation, or it may invoke
Filters, Specialised Adapters, Base Adapters and the Canonical
Interaction Model (CIM). It then returns the resulting interaction to
Moderari Core.

SIE is intended to provide a stable, extensible framework. It must not
contain hard-coded knowledge of individual interaction representations,
models, providers, Specialised Adapters or Filters.

The base implementation therefore defines contracts, discovery,
registration, lifecycle and execution boundaries rather than concrete
protocol or Filter behaviour.

> **SIE knows the structure and contracts of interaction components. It
> does not know the semantics of their implementations.**

------------------------------------------------------------------------

## 2. Core Architectural Boundary

SIE is the entry and exit point for interaction processing inside
Moderari.

``` text
                 MODERARI CORE
                       │
                       │ interaction
                       ▼
              ┌───────────────────┐
              │       SIE         │
              │                   │
              │  determine path   │
              │  invoke Filters   │
              │  invoke Adapters  │
              │  traverse CIM     │
              │  manage loopback  │
              │                   │
              └─────────┬─────────┘
                        │
                        │ resulting interaction
                        ▼
                 MODERARI CORE
```

Moderari Core does not contain Adapter, CIM, Filter or Filter-loopback
orchestration logic. Those responsibilities remain behind the SIE
boundary.

Moderari Core remains responsible for the surrounding execution and
transport responsibilities, including interaction with the selected
provider/model.

------------------------------------------------------------------------

## 3. Foundation Principle: Natural Extensibility

The SIE framework must allow new implementations to become available
without requiring changes to SIE Core.

This applies to:

-   Base Adapters;
-   Specialised Adapters;
-   Filters.

Concrete capabilities enter SIE through discovered, self-describing
implementations.

The framework must not contain hard-coded lists such as supported
protocol names, model families, Filter names or provider names.

A component describes itself through a manifest or equivalent discovery
metadata and implements the appropriate SIE contract in code.

Configuration metadata describes the component. Executable behaviour
remains in the component implementation.

The desired property is:

> **Installing or removing an implementation changes the capabilities
> available to SIE without requiring a change to SIE framework code.**

------------------------------------------------------------------------

## 4. Concepts Known by SIE

SIE understands the structural existence of the following concepts:

-   Representation;
-   Base Adapter;
-   Specialised Adapter;
-   Filter;
-   Effective Model Definition (EMD);
-   Canonical Interaction Model (CIM);
-   Session Execution Plan;
-   session-scoped Execution Instance.

SIE understands the contracts associated with these concepts, but not
the implementation-specific semantics of individual instances.

For example, SIE can know that a Base Adapter:

-   has an identity and version;
-   represents one external interaction representation;
-   can recognise that representation;
-   can translate that representation to CIM;
-   can translate CIM to that representation.

SIE does not need to know what a particular external representation
means or how a particular Base Adapter performs its translation.

------------------------------------------------------------------------

## 5. Component Identity

All discovered executable components require a generic identity
sufficient to distinguish an implementation and its version.

The foundation should provide a generic immutable component identity
which can be used by Base Adapters, Specialised Adapters and Filters.

The identity mechanism must not embed component-specific behaviour or
assumptions.

Exact identity fields will be kept to the minimum demonstrated by
implementation requirements. Version information is required because the
Session Execution Plan and later replay behaviour must be able to
identify the implementation that was selected.

------------------------------------------------------------------------

## 6. Representations

A representation is identified generically. Representation types are not
hard-coded into SIE.

Known representations emerge from discovered Base Adapter
implementations.

For example, if a Base Adapter describing representation `X` is
installed and successfully discovered, `X` becomes a representation
known to SIE. If that implementation is removed, SIE does not retain
hard-coded knowledge of `X`.

Unknown client/provider+model representation is not itself a representation type. It
means that SIE was unable to positively recognise the representation
presented by the client.

This distinction is important:

``` text
Unknown representation ≠ a representation named "unknown"
```

------------------------------------------------------------------------

## 7. Base Adapter Contract

A Base Adapter provides the bridge between one external interaction
representation and CIM.

Conceptually, every Base Adapter supports:

``` text
External Representation
        ↕
    Base Adapter
        ↕
       CIM
```

The Base Adapter contract therefore needs to support, at minimum:

-   component identity;
-   representation identity;
-   representation recognition;
-   external representation → CIM translation;
-   CIM → external representation translation.

The framework does not know the translation rules.

Base Adapters do not know about one another. SIE composes them through
CIM.

### 7.1 Request Direction

For a client using representation A and a provider/model using
representation B:

``` text
CLIENT
Representation A
      │
      ▼
Base Adapter A
      │
      ▼
     CIM
      │
      ▼
Base Adapter B
      │
      ▼
MODEL
Representation B
```

### 7.2 Response Direction

The same components naturally reverse direction:

``` text
MODEL
Representation B
      │
      ▼
Base Adapter B
      │
      ▼
     CIM
      │
      ▼
Base Adapter A
      │
      ▼
CLIENT
Representation A
```

The useful concepts are therefore source representation and destination
representation rather than permanently assigning Adapter A to the client
and Adapter B to the provider.

### 7.3 Same-Representation Short Circuit

Where source and destination representations are already compatible and
no Specialised Adapter is required, SIE does not unnecessarily traverse
CIM.

``` text
Representation A ─────────────► Representation A
                 no CIM bridge
```

CIM is a compatibility bridge, not a mandatory internal transport
representation.

------------------------------------------------------------------------

## 8. Specialised Adapter Contract

A Specialised Adapter handles known deviations associated with a
provider/model or model family while retaining an underlying Base
Representation.

A Specialised Adapter does not define a new Base Representation and does
not own CIM.

Its self-description must provide sufficient information for SIE to
determine when it applies, including the underlying Base Representation
and its provider/model or family applicability.

SIE resolves whether a Specialised Adapter is required according to the
previously established deterministic resolution rules. The Specialised
Adapter itself owns the behaviour needed to correct the interaction.

SIE must not contain model-specific branches.

------------------------------------------------------------------------

## 9. Effective Model Definition (EMD)

The Effective Model Definition is supplied to Moderari by Praebere.

Praebere is responsible for constructing the EMD and is authoritative
for provider/model knowledge. SIE does not construct the EMD, discover
model characteristics, record their provenance or challenge their
values.

SIE treats the EMD as the current provider/model definition supplied by
Praebere.

### 9.1 Filter Requirements

A Filter may declare that particular EMD attributes are required for
that Filter to be available.

SIE performs only the availability test.

Conceptually:

``` text
Filter requires attribute X
          │
          ▼
Does current EMD satisfy X?
          │
       YES / NO
```

Where an EMD requirement is represented as true, false or unknown, SIE
uses that current result for applicability. It does not need to
understand the model, the origin of the information or the operational
meaning of the characteristic.

A Filter owns any interpretation or boundary behaviour associated with
the information it requires. The SIE framework does not implement
Filter-specific rules such as numeric thresholds.

The EMD is therefore an applicability/configuration input to SIE. It is
not model knowledge owned by SIE and does not need to be passed to a
Filter merely because the Filter's declared requirements were satisfied.

### 9.2 No Persistent Compatibility Knowledge

SIE must not persist model-to-Filter or model-to-Adapter compatibility
conclusions as general knowledge.

A future EMD may differ from a previous EMD for the same nominal
provider/model. Availability is therefore evaluated against the current
EMD when configuration is performed and is revalidated when the first
Ask materialises the session execution configuration.

------------------------------------------------------------------------

## 10. Filter Contract

A Filter is a discovered, self-contained implementation of an
intentional interaction transformation.

The Filter contract must provide sufficient self-description for SIE to
determine whether the Filter can participate in the current execution
environment without SIE understanding what the Filter does.

At minimum, a Filter may need to declare:

-   component identity and version;
-   its native interaction representation;
-   required EMD attributes;
-   its execution mode.

The exact manifest/schema will be defined from demonstrated
implementation requirements rather than anticipated future features.

### 10.1 Filter Execution Modes

A Filter may perform a direct transformation, or it may require a
private model interaction as part of its execution.

The framework therefore needs to distinguish structurally between at
least:

``` text
TRANSFORM
MODEL_INTERACTION
```

The names remain implementation choices, but the distinction is
architectural.

A direct transformation completes when the Filter produces its
transformed output.

A model-interaction Filter may produce a private model Ask and require
the corresponding model Response before it can produce its Final Filter
Output.

------------------------------------------------------------------------

## 11. Private Filter Model Interaction

A Filter must not directly communicate with a provider/model.

If a Filter requires a model interaction, control remains with SIE while
Moderari Core provides the actual model execution capability.

Conceptually:

``` text
Filter
   │
   │ private model Ask
   ▼
SIE
   │
   │ provider-compatible interaction
   ▼
Moderari Core
   │
   ▼
Provider / Model
   │
   ▼
Moderari Core
   │
   ▼
SIE
   │
   ▼
same Filter invocation
```

Moderari Core is responsible for transport/execution. SIE remains
responsible for the Filter operation.

The Filter does not need knowledge of Praebere, provider endpoints,
Adapter paths, CIM or other Moderari services.

### 11.1 Filter Execution Identity

Each Filter invocation that requires correlation through an execution
path is assigned a `filter_execution_id` by Moderari/SIE.

This allows a private model Response to be returned to the same Filter
invocation and distinguishes that interaction from the main
client-visible model interaction.

The `filter_execution_id` is session/execution correlation, not
persistent model knowledge.

------------------------------------------------------------------------

## 12. Filter Loopback

Filter loopback is owned by SIE and has one authoritative implementation
location.

Moderari Core does not contain loopback rules.

A Filter operates in its declared native representation. If its Final
Filter Output must be incorporated into an interaction whose client
representation differs, SIE performs the mandatory translation back into
the client representation.

``` text
Filter Final Output
       │
       │ Filter native representation
       ▼
      SIE
       │
       ├── same as client representation
       │       └── no translation
       │
       └── different representation
               │
               ▼
          Base Adapter
               │
               ▼
              CIM
               │
               ▼
          Base Adapter
               │
               ▼
        client representation
```

Loopback remains part of Filter execution even where normal
client-to-provider Adapter translation has been disabled according to
the established Adapter rules.

The Filter itself does not manage CIM or choose Adapter paths.

------------------------------------------------------------------------

## 13. Moderari Core ↔ SIE Contract

SIE requires an explicit contract with Moderari Core.

From Moderari Core's perspective, the normal pattern is:

``` text
interaction + session_id
          │
          ▼
         SIE
          │
          ▼
processed interaction
          │
          ▼
    Moderari Core
```

Adapter traversal does not require a separate request/response direction
flag. For a cross-protocol path, the resolved Base Adapter pair supplies
the external-representation-to-CIM and CIM-to-external-representation
bridge. For a same-protocol path with an applicable Specialised Adapter,
one Specialised Adapter is the effective Adapter for the complete
interaction and architecturally occupies both Adapter roles. The payload
protocol therefore does not need to distinguish between a Base Adapter
and a Specialised Adapter at runtime.

For Filters requiring private model interactions, the Core/SIE contract
must also provide SIE with a way to request model execution and receive
the corresponding Response without transferring Filter orchestration to
Core.

------------------------------------------------------------------------

## 14. Configuration Lifecycle

SIE participates in two distinct phases before normal repeated
execution.

### 14.1 Configuration-Time Availability

After model selection, Praebere supplies the current EMD.

SIE uses the current EMD and the currently discovered components to
determine which Filters are presently available for selection.

These availability conclusions are transient. They are not stored as
persistent provider/model compatibility knowledge.

### 14.2 First-Ask Validation and Materialisation

The first Ask supplies information that was not available during initial
configuration, particularly the client's interaction representation.

At first Ask, SIE revalidates the selected configuration against the
current EMD and resolves the actual execution path, including as
applicable:

-   client representation;
-   provider representation;
-   Base Adapter path;
-   Specialised Adapter path;
-   selected Filters;
-   Filter native representations;
-   mandatory Filter loopback paths.

This is the point at which the immutable Session Execution Plan is
finalised according to the previously established Moderari architecture.

------------------------------------------------------------------------

## 15. Session-Scoped Execution Instance

SIE must not receive and resolve the complete EMD for every Ask.

Once first-Ask validation and Session Execution Plan materialisation are
complete, SIE creates a minimal, session-scoped, on-the-fly Execution
Instance associated with the `session_id`.

The Execution Instance is the hot runtime machinery for that session.

It is distinct from the Session Execution Plan:

``` text
Session Execution Plan
    authoritative, immutable description

Execution Instance
    minimal, hot, session-scoped runtime machinery
```

The Execution Instance should contain only what SIE requires to execute
the already-resolved configuration efficiently. It must not become a
duplicate representation of the Session Execution Plan.

Where useful, it may hold resolved live component instances or
references so that SIE does not repeatedly perform discovery,
applicability checks or Adapter-path resolution.

Conceptually:

``` text
FIRST ASK
    │
    ├── validate current EMD
    ├── recognise client representation
    ├── resolve components and paths
    ├── finalise immutable Session Execution Plan
    │
    ▼
create Execution Instance
    │
    └── attach to session_id
```

Subsequent execution becomes:

``` text
Moderari Core
      │
      │ session_id + interaction
      ▼
SIE
      │
      ▼
Execution Instance[session_id]
      │
      ▼
execute already-resolved path
      │
      ▼
Moderari Core
```

No repeated EMD lookup, component discovery, Filter applicability
evaluation or Adapter resolution is required for the normal session
path.

### 15.1 Session Ownership

The Execution Instance does not mean that SIE owns the Lumen session.
Session identity/lifecycle remains outside SIE according to the
established service boundaries.

SIE owns only the runtime interaction-processing machinery associated
with that session.

### 15.2 Lifetime

``` text
session created
      │
      ▼
configuration
      │
      ▼
first Ask
      │
      ├── Session Execution Plan finalised
      └── Execution Instance created
      │
      ▼
subsequent Asks / Responses
      │
      └── reuse Execution Instance
      │
      ▼
session end
      │
      └── destroy Execution Instance
```

The Execution Instance is disposable. The Session Execution Plan is the
authoritative configuration/evidence needed for assurance and later
replay.

A new session receives a new current EMD and resolves its execution
environment independently.

> **SIE remembers what this session has resolved; it does not remember
> what a model is capable of.**

------------------------------------------------------------------------

## 16. SIE Execution Direction

SIE processes interactions in both directions.

### Client → Provider

``` text
Client
  │
  ▼
Moderari Core
  │
  ▼
SIE Execution Instance
  │
  ├── selected Filters
  ├── private Filter model interaction where required
  ├── mandatory Filter loopback where required
  ├── Specialised Adapter where required
  ├── Base Adapter / CIM path where required
  └── no transformation where appropriate
  │
  ▼
Moderari Core
  │
  ▼
Provider / Model
```

### Provider → Client

``` text
Provider / Model
  │
  ▼
Moderari Core
  │
  ▼
SIE Execution Instance
  │
  ├── Specialised Adapter where required
  ├── Base Adapter / CIM path where required
  └── no transformation where appropriate
  │
  ▼
Moderari Core
  │
  ▼
Client
```

Filters remain to-model transformations under the current architecture
and do not transform the normal client-visible model Response.

------------------------------------------------------------------------

## 17. Canonical Interaction Model

CIM is the fixed interoperability centre used by Lumen-controlled Base
Adapters.

Unlike representation implementations, Specialised Adapters, Filters and
EMD attributes, CIM is not an arbitrary extension surface.

``` text
Representation A
      ↕
Base Adapter A
      ↕
     CIM
      ↕
Base Adapter B
      ↕
Representation B
```

CIM provides the shared semantic contract that allows Base Adapters to
interoperate without knowing about one another.

Third-party component extensibility must not result in arbitrary
extension of CIM. CIM remains Lumen-defined and versioned according to
the existing architecture decisions.

------------------------------------------------------------------------

## 18. Discovery and Registration

SIE must provide a common discovery and registration mechanism for
executable components.

The initial framework must support discovery of:

-   Base Adapters;
-   Specialised Adapters;
-   Filters.

Discovered components are self-describing. SIE uses their metadata to
register them and determine when they are structurally available.

The framework should be capable of proving extensibility using trivial
test/dummy components that live outside SIE Core. Adding those
components should require no modification to the framework itself.

The discovery mechanism must also enforce established invariants such as
invalid duplicate component registrations where applicable.

Exact packaging and manifest formats will be defined during
implementation rather than over-specified in this baseline.

------------------------------------------------------------------------

## 19. Error Philosophy

The initial SIE framework should define only errors demonstrated by
current requirements.

It must not attempt to predict a complete future error taxonomy.

As implementation proceeds, new error conditions will be considered
individually and incorporated into a coherent domain-error rule based on
evidence from the working system.

A distinction must be preserved between:

-   expected execution outcomes, such as a component being unavailable
    or a known interaction path being incompatible; and
-   broken invariants or invalid installation/configuration states,
    which may require an exception or startup failure.

Normal interaction flow should not be implemented through broad
exception handling.

------------------------------------------------------------------------

## 20. Persistence and Framework Dependencies

The SIE foundation is an internal interaction domain and execution
framework.

It should remain independent of unnecessary infrastructure dependencies,
including:

-   FastAPI;
-   HTTP transport;
-   MongoDB;
-   Praebere implementation details;
-   Pontis implementation details;
-   provider-specific clients.

Standard Python domain structures should be preferred for the
foundation. Transport/persistence models may be introduced at their
appropriate boundaries later.

The session-scoped Execution Instance is runtime state and is not itself
a persistence model.

------------------------------------------------------------------------

## 21. Initial Framework Boundary

The initial internal package should reflect framework responsibilities
rather than concrete implementations.

A conceptual starting boundary is:

``` text
interaction/
│
├── components/
│   ├── identity
│   ├── manifest
│   └── discovery
│
├── representations/
│   └── contract
│
├── adapters/
│   ├── base contract
│   └── specialised contract
│
├── filters/
│   └── contract
│
├── cim/
│   └── contract/schema
│
├── emd/
│   └── structural contract
│
├── execution/
│   ├── execution instance
│   └── session/runtime lifecycle
│
└── errors
```

Exact Python filenames and class names should be allowed to emerge
during implementation. The important constraint is the responsibility
boundary.

There is no current requirement for a generic `decisions.py`
abstraction. Component availability is fundamentally a current yes/no
evaluation against the applicable rules, with first-Ask revalidation
before execution materialisation.

------------------------------------------------------------------------

## 22. IP-01 Foundation Acceptance Principle

The foundation is successful when the following can be demonstrated
without changing SIE Core:

1.  A dummy Base Adapter can be installed and discovered.
2.  Its representation becomes known through discovery rather than
    hard-coded framework knowledge.
3.  A dummy Specialised Adapter can be installed and registered against
    the appropriate structural applicability information.
4.  A dummy Filter can be installed and discovered.
5.  The Filter can declare EMD requirements.
6.  SIE can determine current Filter availability from an arbitrary
    current EMD without understanding the Filter or model semantics.
7.  At first Ask, SIE can recognise/revalidate the interaction
    environment and materialise a session-scoped Execution Instance.
8.  The Execution Instance can hold resolved component
    instances/references and execute without repeated discovery or EMD
    resolution.
9.  A Base Adapter can translate representation → CIM and CIM →
    representation through the common contract.
10. SIE can compose two Base Adapters through CIM without either Adapter
    knowing about the other.
11. A model-interaction Filter can request a private model interaction
    through the SIE/Core boundary and receive the corresponding Response
    back into the same Filter invocation.
12. Filter loopback is performed in one authoritative location inside
    SIE.
13. Ending the session disposes of the hot Execution Instance without
    creating persistent provider/model compatibility knowledge.

The central implementation test is therefore:

> **Given discovered components, a current EMD, an immutable session
> configuration and an interaction, can SIE determine and execute the
> required interaction path without hard-coded knowledge of the concrete
> Adapter, Filter, provider, model or external representation
> involved?**

------------------------------------------------------------------------

## 23. Architectural Summary

The Standalone Interaction Engine is a framework and execution boundary,
not a catalogue of model or protocol behaviour.

Its responsibilities are to:

-   discover self-contained interaction components;
-   understand their contracts and structural declarations;
-   use the current EMD to determine component availability without
    owning model knowledge;
-   recognise client representation at first Ask;
-   resolve and materialise the interaction-processing configuration;
-   create a minimal hot Execution Instance attached to the session;
-   invoke Filters and manage private Filter model interactions;
-   own mandatory Filter loopback;
-   invoke Specialised Adapters;
-   compose Base Adapters through CIM when representations differ;
-   short-circuit unnecessary translation;
-   process provider-bound and client-bound interactions;
-   dispose of runtime execution state at session end.

It explicitly does **not** own:

-   provider/model knowledge;
-   construction or provenance of the EMD;
-   persistent model↔component compatibility conclusions;
-   provider/model transport;
-   Lumen session ownership;
-   the semantics of individual Filters or Adapters.

This separation provides the foundation for a naturally extensible
Moderari interaction engine while preserving deterministic session
execution and the existing Lumen service boundaries.

## 02026-10-06 Adapter Execution Clarification

### Base Adapter IN/OUT Model

A Base Protocol Adapter owns one external protocol and the bidirectional
mapping between that protocol and CIM. Adapter direction is defined
relative to CIM, not relative to client/provider or request/response:

``` text
IN:  external protocol -> CIM
OUT: CIM -> external protocol
```

For two resolved effective Adapters A and B, SIE uses one symmetric
execution rule:

``` text
Protocol A payload -> Adapter A IN -> CIM -> Adapter B OUT -> Protocol B payload
Protocol B payload -> Adapter B IN -> CIM -> Adapter A OUT -> Protocol A payload
```

The representation of the incoming payload therefore determines which
effective Adapter operates IN; the other operates OUT. SIE does not
require separate client-to-provider and provider-to-client Adapter
algorithms.

### Base Protocol Adapters versus Specialised Adapters

> **Base Protocol Adapters provide cross-protocol interoperability
> through CIM. Specialised Adapters provide provider/model-specific
> specialisation only within a same-protocol interaction.**

A Specialised Adapter is based on one Base Protocol Adapter and
therefore inherits that Base Adapter's protocol. It does not define a
new protocol and does not provide a cross-protocol bridge.

A Specialised Adapter is eligible only when:

``` text
client protocol
    ==
Specialised Adapter Base Protocol
    ==
provider+model protocol

AND

provider + exact model/family match
```

Provider/model matching alone is not sufficient.

For the current Qwen case, the Specialised Adapter is an OpenAI-based
specialisation for the demonstrated provider/model combination. It may
change only specific model-produced material, such as tool-call
representation, and may require no transformation in the opposite
direction. This asymmetry is valid.

Conceptually, when applicable:

``` text
OpenAI Client
      <->
Qwen/OpenAI Specialised Adapter
      <->
     CIM
      <->
OpenAI Base Protocol Adapter
      <->
Ollama / Qwen2.5-Coder
```

The same Specialised Adapter is not eligible for an Anthropic client
merely because provider+model matches. That is a cross-protocol
interaction and must use Base Protocol Adapters through CIM, or be
rejected if the known provider/model deviation makes that Base-only path
unsafe. SIE must never reuse a same-protocol Specialised Adapter outside
its Base Protocol boundary.

### Resolution Consequence

At first Ask, after client protocol and provider+model protocol are
known:

1.  If the protocols are the same, SIE may resolve an exact then family
    Specialised Adapter based on that same Base Protocol.
2.  If no applicable Specialised Adapter exists, the same-protocol path
    may use the established pass-through/no-Adapter path.
3.  If the protocols differ, Specialised Adapters are not eligible; SIE
    resolves Base Protocol Adapter A -\> CIM -\> Base Protocol Adapter
    B.
4.  If the required cross-protocol Base path is unavailable or unsafe,
    SIE rejects rather than guessing.

### Execution Instance Consequence

The hot session-scoped Execution Instance needs the resolved effective
Adapter pair (and any selected Specialised Adapter implementation), not
separate request and response Adapter paths. At runtime the incoming
payload's protocol selects the IN side and the opposite resolved Adapter
is the OUT side.

## 02026-10-06 Specialised Adapter Pair-Replacement Clarification

### Architectural Rule

A Specialised Adapter is a same-protocol specialisation of an existing
Base Protocol Adapter. It is eligible only when the recognised client
protocol, the Specialised Adapter's Base Protocol, and the
provider+model protocol are the same, in addition to the required
provider/model or family match.

When such a Specialised Adapter is resolved, it is architecturally the
effective **Adapter A and Adapter B** for that same-protocol
interaction. SIE does not compose the Specialised Adapter with a second
Base Adapter across CIM.

Conceptually:

``` text
same protocol, no specialisation

Protocol X payload --------------------------> Protocol X payload
                    pass-through
```

``` text
same protocol, Specialised Adapter resolved

Protocol X payload
        |
        v
+---------------------------+
|   Specialised Adapter     |
|                           |
|   effective Adapter A     |
|   effective Adapter B     |
|                           |
|   may use Base Adapter    |
|   and CIM internally      |
+---------------------------+
        |
        v
Protocol X payload
```

By contrast, differing protocols continue to use the ordinary Base
Adapter bridge:

``` text
Protocol A payload
        |
        v
Base Adapter A IN
        |
        v
       CIM
        |
        v
Base Adapter B OUT
        |
        v
Protocol B payload
```

Therefore:

> **Base Protocol Adapter pairs plus CIM provide cross-protocol
> interoperability. A resolved Specialised Adapter replaces both sides
> of the Adapter pair for a same-protocol specialised interaction.**

### Specialised Adapter Execution Semantics --- Current Interpretation

The exact internal implementation of a Specialised Adapter remains
deliberately open for IP-01 implementation and testing.

The current working interpretation is:

1.  every invocation enters the Specialised Adapter as a payload in its
    inherited Base Protocol;
2.  the Specialised Adapter may recognise and correct
    provider/model-specific deviations while processing that incoming
    payload;
3.  the corrected semantic interaction may be represented internally as
    CIM using the inherited Base Adapter contract;
4.  the inherited Base Adapter materialisation can then return the
    normal Base Protocol payload; and
5.  if the incoming payload requires no specialised correction, the
    implementation may preserve/pass through the Base Protocol
    interaction as appropriate.

This is intended to allow one Specialised Adapter implementation to
handle deviations observable in either interaction flow without
requiring SIE or the Adapter manifest to describe `to_model`,
`from_model`, client-side or provider-side transformation directions.

For the current Qwen case, the known deviation is present in
model-produced OpenAI-compatible payload material, particularly
textual/tool-call material. A Qwen Specialised Adapter can recognise
that deviation when the protocol payload enters the Adapter,
canonicalise/correct it, and rely on the Base OpenAI behaviour to
materialise the normal OpenAI-compatible payload.

The implementation mechanism is **not yet frozen**. In particular, IP-01
must test how specialised IN behaviour, inherited Base IN/OUT behaviour,
internal CIM use and pass-through interact so that an internal
conversion cannot accidentally invoke an opposing specialised override
or reverse a correction.

The architecture is frozen only at the following level:

> **A Specialised Adapter is one same-protocol effective Adapter
> occupying the architectural roles that Adapter A and Adapter B would
> otherwise occupy. Its internal use of Base Adapter operations and CIM
> is an implementation concern and remains open to further
> interpretation during IP-01.**

A Specialised Adapter does not expose its internal CIM traversal as an
inter-Adapter CIM boundary, and SIE must not route CIM from another
Adapter into a Specialised Adapter. Cross-protocol interaction is
handled by Base Protocol Adapters, not by Specialised Adapters.
