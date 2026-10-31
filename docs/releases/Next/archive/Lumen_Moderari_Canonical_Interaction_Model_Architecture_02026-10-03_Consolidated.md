# Lumen Moderari Canonical Interaction Model Architecture

## Internal Semantic Mapping for the Moderari Adapter Framework

**Status:** Architecture proposal for review and further discussion\
**Scope:** Canonical Interaction Model (CIM) only\
**Current implementation commitment:** None beyond defining the
architecture required by the Moderari Adapter Framework

------------------------------------------------------------------------

# 1. Purpose

The **Canonical Interaction Model (CIM)** is Moderari's internal
semantic mapping model for the Adapter Framework.

Its purpose is to prevent Lumen from requiring a separate translator for
every possible pair of client and provider interaction representations.

Without a canonical model:

``` text
OpenAI      <-> Anthropic
OpenAI      <-> Meta
OpenAI      <-> Gemini
Anthropic   <-> Meta
Anthropic   <-> Gemini
Meta        <-> Gemini
...
```

With the CIM:

``` text
OpenAI      <-> CIM
Anthropic   <-> CIM
Meta        <-> CIM
Gemini      <-> CIM
Other       <-> CIM
```

The CIM is not a network protocol and is not exposed as a new Lumen
protocol.

> **The CIM is an internal semantic data model used by enabled Moderari
> Adapters to map equivalent interaction concepts between different
> external representations.**

------------------------------------------------------------------------

# 2. CIM Is Part of the Adapter Framework

The CIM is instantiated when Adapter Framework translation or client-representation materialisation requires it. Normal same-representation provider execution still does not introduce a CIM round-trip merely because CIM exists.

``` text
ADAPTERS ENABLED

Client Representation
        │
        ▼
     Adapter
        │
        ▼
       CIM
        │
        ▼
     Adapter
        │
        ▼
Provider Representation
```

When Adapter processing is Disabled, Moderari does not introduce CIM
normalization merely because the CIM exists.

``` text
ADAPTERS DISABLED

Client Representation
        │
        ▼
     Moderari
        │
        ▼
Provider Representation
```

Any interaction attempted with Adapters disabled therefore depends on
the client/provider representations already being mutually usable.

The CIM is not the universal internal representation of Vanilla Moderari.

> **No required Adapter translation or client-representation materialisation means no CIM runtime path.**

------------------------------------------------------------------------

### 3. CIM Is Independent of Filters

Moderari Filters do not construct, consume or operate on the CIM. The CIM is internal to the Adapter Framework.

A Filter produces new interaction material in **one external interaction representation supported by Lumen** that the Filter knows how to construct. The Filter is not required to understand or produce every interaction representation supported by Lumen.

The Filter-produced material must ultimately be incorporated into the session context in the recognised **client interaction representation**.

```
Filter
   │
   │ produces material in one
   │ supported external representation
   ▼
Filter Output Representation
   │
   ├── same as client representation
   │         │
   │         └──────────────► incorporate directly
   │
   └── different
             │
             ▼
      Adapter Framework
             │
             │ translation via CIM
             ▼
      Client Representation
             │
             ▼
      incorporate into
       session context
```

Where translation is required, the Adapter Framework may internally use the CIM and the appropriate Lumen Base Representation Adapters. This is **loopback translation** and is invisible to the Filter.

Examples include:

```
System Prompt Filter
    produces system-context material
    in its supported output representation

Cognition Filter
    produces conversational-context material
    in its supported output representation
```

The responsibilities therefore remain distinct:

```
Filter
    intentionally changes interaction content
    presented to the model

Adapter
    changes representation while preserving
    the intended interaction

CIM
    provides the internal semantic bridge
    used by Base Representation Adapters
```

A Filter does not need to understand how a different external representation expresses the material it creates. For example, a System Prompt Filter that produces its insertion in OpenAI representation does not need to understand how Anthropic represents equivalent system context. Where the recognised client representation is Anthropic, the Adapter Framework performs the required loopback translation before the material is incorporated into the session context.

Filter operation therefore requires a **recognised client interaction representation**. In addition, a Filter Profile is selectable only when the provider interaction representation is also known, because Moderari must be able to establish the complete transformed execution path to the provider.

Filter Profile model applicability is evaluated before the first Ask from
Praebere-supplied model facts and does not require the client representation to
be known. If Filters are selected, the first Ask must positively establish the
client representation and a compatible representation path before Filter
execution. An Unknown client representation with No Filter Profile may use the
normal pass-through path; an Unknown client representation with selected
Filters is rejected.

The detailed architecture of Cognition and any future context-management Filters is intentionally deferred until the relevant Moderari implementation is revisited.

------------------------------------------------------------------------

# 4. Semantic Mapping Model

The CIM represents **semantic interaction concepts** derived from documented
external interaction contracts, not the field names chosen by a particular
protocol or quirks observed in a particular provider/model implementation.

The initial CIM should be grounded in the published OpenAI and Anthropic
interaction schemas. Those published contracts provide the first concrete
semantic source material from which common CIM concepts are defined.

> **The CIM is defined against documented external interaction semantics, not
> observed model behaviour.**

For example, different representations may express a tool invocation
differently:

``` text
OpenAI                         CIM                       Anthropic
────────────────              ──────────────            ────────────────
tool_calls[]        ───────►   tool_call      ◄───────   tool_use

id                  ───────►   tool_call.id   ◄───────   id

function.name       ───────►   tool.name      ◄───────   name

function.arguments  ───────►   tool.arguments ◄───────   input
```

A Meta or future representation can map its equivalent concept to the
same CIM semantic object.

The Adapter therefore does not need to know about every other Adapter.

``` text
OpenAI Adapter
      │
      ▼
CIM tool_call
      ▲
      │
Anthropic Adapter
```

The semantic concept is stable even when its external representation
differs.

------------------------------------------------------------------------

# 5. CIM Schema

The CIM requires an explicit schema.

The schema defines:

``` text
semantic object types
attributes belonging to those objects
attribute types
required/optional attributes
relationships between objects
extension points
schema identity
schema version
```

An illustrative structure might contain:

``` yaml
cim:
  schema: lumen.moderari.cim
  version: 1.0

  objects:

    message:
      attributes:
        role: string
        content: content

    tool:
      attributes:
        name: string
        description: string
        input_schema: object

    tool_call:
      attributes:
        id: string
        name: string
        arguments: object

    tool_result:
      attributes:
        call_id: string
        content: content
```

This is illustrative only. It is **not yet the final CIM schema**.

The schema should initially contain only concepts required by supported
Adapter use cases rather than attempting to model every feature exposed
by every AI protocol.

------------------------------------------------------------------------

# 6. Schema Evolution

The CIM cannot be assumed to remain static.

New interaction representations may introduce semantic concepts that the
current CIM does not express.

Examples might include:

``` text
new content-part types
new tool interaction structures
reasoning-related structures
citations
artifacts
computer actions
multimodal content
future concepts not presently known
```

The CIM must therefore be **versioned and evolvable**.

Conceptually:

``` text
CIM 1.0
   │
   ├── message
   ├── tool
   ├── tool_call
   └── tool_result

CIM 1.1
   │
   └── additional compatible semantic concept

CIM 2.0
   │
   └── incompatible schema evolution
```

The exact versioning rules remain to be designed.

A CIM schema update is an Adapter-framework architecture change, not an
arbitrary per-session user configuration change.

> **The CIM is extensible and versioned; it is not casually mutable at
> runtime.**

------------------------------------------------------------------------

# 7. Adapter-to-CIM Contract

Each Adapter must declare the CIM schema version or versions with which
it is compatible.

Conceptually:

``` yaml
adapter:
  id: openai-chat-completions
  version: 1.0.0

  external_representation:
    id: openai-chat-completions

  cim:
    supported_versions:
      - "1.x"
```

The precise manifest syntax remains an implementation-design decision.

An Adapter is responsible for translating between:

``` text
External Representation
        ⇅
Supported CIM Schema
```

It is not responsible for translating directly to another external
representation.

------------------------------------------------------------------------

# 8. Mapping Is Not Necessarily Declarative

Some mappings may be simple enough to describe declaratively.

For example:

``` yaml
tool_call:
  external:
    collection: tool_calls
    id: id
    name: function.name
    arguments: function.arguments

  canonical:
    object: tool_call
    id: id
    name: name
    arguments: arguments
```

Another representation might declare:

``` yaml
tool_call:
  external:
    type: tool_use
    id: id
    name: name
    arguments: input

  canonical:
    object: tool_call
    id: id
    name: name
    arguments: arguments
```

However, the architecture must not assume that every protocol
translation can be expressed as field-to-field configuration.

Some representations may require executable logic for:

``` text
nested structures
content-part conversion
conditional fields
stream assembly
tool-call reconstruction
validation
normalization
```

Therefore:

> **The CIM schema defines the semantic target. The Adapter
> implementation determines how its external representation is mapped to
> and from that target.**

------------------------------------------------------------------------

# 9. Minimum Semantic Model

The CIM should be deliberately smaller than the union of all external
protocols.

The initial semantic vocabulary should be driven by actual Adapter
requirements.

Candidate concepts include:

``` text
Interaction
│
├── system context
│
├── messages
│     ├── role
│     └── content
│
├── tool definitions
│     ├── name
│     ├── description
│     └── input schema
│
├── tool calls
│     ├── call identity
│     ├── tool identity
│     └── arguments
│
├── tool results
│     ├── call identity
│     └── result content
│
└── execution/request options required for representation compatibility
```

This list is provisional.

The first real schema should be derived from the documented external
representations Lumen needs to bridge, beginning with the published OpenAI
and Anthropic interaction contracts.

Provider/model-specific deviations, including the existing Qwen compatibility
case, must not redefine the CIM. Such deviations belong to specialised
Adapters derived from the relevant base representation Adapter.

------------------------------------------------------------------------

# 10. Information Preservation

Information preservation in the CIM applies to translation between
**conforming published interaction representations**.

Consider:

``` text
Published Representation A
    X
    Y
    Z

CIM
    X
    Y
    Z

Published Representation B
    X
    Y
```

If `Z` is a documented semantic concept in Representation A but has no
equivalent in Representation B, Lumen has encountered a genuine
representation-interoperability limitation.

That condition belongs to the CIM/representation-Adapter boundary and must not
be silently hidden.

The architecture therefore requires a policy for documented semantics that
are:

``` text
present in a supported published source representation
represented by the CIM
not representable by the supported published target representation
```

The exact policy remains to be designed. It may ultimately reject the
translation, report the incompatibility, or explicitly permit a documented
loss.

The important requirement is:

> **A base representation Adapter must not silently discard documented
> published semantics while claiming a representation-preserving
> translation.**

Provider/model-specific deviations are a different problem and are explicitly
outside this CIM preservation responsibility.

If a provider/model that declares a published representation instead emits a
non-standard form, any correction belongs to a specialised Adapter derived
from the relevant base representation Adapter.

``` text
Published contract
    X Y Z
      │
      ▼
Provider/model deviation
    X Y Q
      │
      ▼
Specialised Adapter
    Q -> Z
```

The author of that specialised Adapter owns the correctness and any lossiness
of `Q -> Z`. The CIM must not be expanded with `Q` merely to accommodate the
deviation.

------------------------------------------------------------------------

# 11. Lossy Translation

There are two distinct cases and they must not be conflated.

## 11.1 Published-Representation Interoperability Loss

``` text
Published Representation A
        │
        ▼
       CIM
        │
        ▼
Published Representation B
        │
        └── cannot express a documented source semantic
```

This is a Lumen Adapter/CIM interoperability concern because both sides are
being interpreted according to their published contracts.

Semantic loss in this case must not be silent. The exact reject/report/permit
policy remains an implementation-design decision.

## 11.2 Provider/Model-Specific Override Loss

``` text
Declared published representation
        │
Provider/model deviates
        │
        ▼
Specialised Adapter override
        │
        ▼
Base representation semantics
```

This is not a CIM schema problem.

A specialised Adapter exists specifically to deal with the provider/model
difference. The author of that Adapter determines how the deviation is mapped
back into the base published representation and therefore owns any
approximation or information loss introduced by that mapping.

Lumen Core must not respond to such a deviation by silently changing the CIM,
changing the published base Adapter contract, or inventing a correction.

> **Published-schema loss is an interoperability concern. Loss introduced by
> a provider/model-specific override is the specialised Adapter author's
> responsibility.**

------------------------------------------------------------------------

# 12. Representation Authority

The CIM does not discover or identify external representations.

Client representation recognition belongs to the Moderari Adapter Framework.
Pontis transports the request and session identity to Moderari but does not
need to classify the AI interaction representation.

Moderari's Representation Resolver uses the request boundary and the
recognition/validation knowledge supplied by installed representation Adapters
to establish the client representation. If it cannot be positively
distinguished, it remains Unknown.

Provider representation remains authoritative Praebere provider knowledge.

``` text
Client request transport
        owner: Pontis

Client representation recognition
        owner: Moderari Adapter Framework

Provider representation
        source: Praebere

Model characteristics
        source: Praebere

Canonical interaction semantics
        owner: CIM

Representation translation
        owner: Moderari Adapters
```

Recognition and canonical semantics are deliberately separate:

``` text
External Request
      │
      ▼
Representation Resolver
      │ recognised representation
      ▼
Adapter
      │
      ▼
CIM
```

The Representation Resolver may inspect defined request-boundary evidence such
as endpoint/path, relevant headers and conformance with a published request
structure. It must not guess from ambiguous or undocumented payload features.

> **The Adapter Framework identifies the representation; the CIM defines the
> canonical semantics.**

# 13. Adapter Resolution and CIM

Adapter discovery and selection are responsibilities of the Moderari Adapter
Framework, not of the CIM itself.

Moderari maintains an internal Adapter Registry built from discovered Adapter
extensions. Praebere supplies provider/model/representation facts; it does not
supply Adapter identities.

``` text
Pontis
  │ client representation
  ▼
Moderari Adapter Resolver
  ▲
  │ provider + model + provider representation
Praebere

Moderari Adapter Resolver
  │
  └── Adapter Registry
```

For provider-side resolution, a specialised Adapter may declare the existing
`provider` and `model` matching values. The `model` value may be either an
exact provider model identifier or the family portion of that identifier.

Moderari derives the family from the actual model identifier as the portion
before the first `:`.

``` text
actual model: qwen2.5-coder:14b

exact:  qwen2.5-coder:14b
family: qwen2.5-coder
```

The deterministic resolution order is:

``` text
provider + exact model
        │ no match
        ▼
provider + derived model family
        │ no match
        ▼
client representation == provider representation?
        │
       YES ─────────────────────────► no Adapter
        │
        NO
        ▼
generic representation Adapter(s) required to cross the boundary
        │ unavailable
        ▼
no Adapter
```

The same-representation test deliberately prevents unnecessary canonical
translation. If client and provider already use the same interaction
representation and no specialised Adapter applies, there is no representation
boundary for CIM to bridge and Moderari uses the No Adapter path.

A specialised provider/model or provider/family Adapter is resolved before
this test because it exists to handle a known deviation from the nominal
provider representation.

Only one Adapter may occupy a particular `provider + model` registry key.
Duplicate keys are invalid configuration rather than a precedence problem.

The exact-model and family matches are alternatives, not stages. Moderari
selects one effective specialised Adapter. A provider/model-specific
specialisation does not form an additional correction stage after the base
representation Adapter.

Conceptually:

``` text
Client Representation
        │
        ▼
effective client representation Adapter
        │
        ▼
       CIM
        │
        ▼
one effective provider representation Adapter
        │
        ▼
Provider Representation
```

The effective provider-side result may therefore be:

``` text
exact provider/model specialisation
or
provider/family specialisation
or
no Adapter when client/provider representations already match
or
generic published-contract representation translation where representations differ
or
no Adapter when required translation cannot be resolved
```

A specialised Adapter derives from/reuses the relevant base representation
Adapter and overrides only the known deviations. The exact implementation
mechanism for specialisation remains deliberately outside the CIM
architecture.

# 14. CIM Runtime Instance

The CIM schema and a CIM runtime instance are different things.

``` text
CIM Schema
    definition of permitted semantic structures
    versioned
    maintained as part of the Adapter framework

CIM Runtime Instance
    one interaction represented using that schema
    transient
    created only while an enabled Adapter path requires it
```

A runtime instance is not intended to become a new persistent
conversation format.

It exists to allow one Adapter to normalize an interaction and another
Adapter to materialize the same semantics into another representation.

------------------------------------------------------------------------

# 15. CIM and Vestigare

CIM representation is not the Vestigare conversation record.

Vestigare records the interaction at the Client ↔ Lumen boundary in the client interaction representation
according to the agreed trace boundary.

CIM is an internal compatibility representation used while Adapters are
enabled.

``` text
Vestigare
    raw conversation / execution metadata

Moderari Adapter Framework
    transient CIM representation
```

The existence of a CIM representation must not cause Vestigare to record
a second canonical copy of the conversation.

Adapter/CIM version information may be retained as execution metadata
where useful, but CIM runtime instances are not the raw trace.

------------------------------------------------------------------------

# 16. Current Qwen Case

The current Qwen tool-call translator is the first concrete example of a
provider/model-specific deviation from a surrounding representation contract.

It must not cause Qwen-specific behaviour to become part of the CIM.

Conceptually:

``` text
Published OpenAI Adapter
        │
        │ complete OpenAI <-> CIM mapping
        │
        └── Qwen/OpenAI Specialisation
                overrides only known Qwen/provider deviations
```

The Qwen specialisation remains an OpenAI representation Adapter for CIM
purposes. It changes only the behaviour required to accommodate the known
provider/model peculiarity.

Where the same behaviour applies across provider model variants such as
`qwen2.5-coder:*`, the Adapter may register `model: qwen2.5-coder` and be
selected through Moderari's derived-family lookup. A more specific
`provider + exact model` registration takes precedence automatically.

If no specialised Adapter is installed or applicable, Lumen does not silently
teach the CIM about the deviation. The base representation contract remains
authoritative and the provider/model's actual response remains observable
evidence.

# 17. Open Design Questions

The following questions are intentionally left open for review:

1.  What is the minimum CIM 1.0 semantic object set?
2.  What schema technology should define the CIM?
3.  What are the exact schema-version compatibility rules?
4.  How are extension/opaque fields represented?
5.  How does an Adapter declare which CIM concepts it supports?
6.  How is semantic loss detected and reported?
7.  What implementation mechanism should realise code reuse/overrides
    between a specialised Adapter and its base representation Adapter?
8.  Which execution/request options genuinely belong in CIM rather than
    Adapter-local state?
9.  How should streaming interactions be represented, if they need
    canonical representation at all?
10. What exact OpenAI and Anthropic published-schema concepts belong in the
    initial CIM?

These should be resolved from concrete interoperability requirements
rather than by attempting to anticipate every future AI protocol.

------------------------------------------------------------------------

# 18. Architectural Invariants

The current CIM architecture establishes the following invariants:

1.  **CIM is an internal Moderari Adapter-framework data model, not a
    Lumen network protocol.**
2.  **CIM is used only when Adapter translation or client-representation materialisation requires a canonical semantic instance.**
3.  **Filters do not operate on CIM.**
4.  **External representations map to CIM semantic concepts rather than
    directly to every other external representation.**
5.  **The CIM has an explicit, versioned and evolvable schema.**
6.  **Adapters declare compatibility with CIM schema versions.**
7.  **Adapter mapping may be executable and is not limited to
    declarative field mapping.**
8.  **CIM/base representation translation must not silently discard
    documented published semantics.**
9.  **Loss between conforming published representations must not occur
    silently; loss introduced by a provider/model-specific override belongs
    to the specialised Adapter.**
10. **Pontis transports the client request/session; Moderari's Adapter
    Framework determines client representation; Praebere supplies provider
    representation and model truth; Moderari performs compatibility
    resolution.**
11. **CIM runtime instances are transient and do not replace the
    Vestigare raw conversation trace.**
12. **The CIM should grow from demonstrated interoperability requirements
    rather than attempt to model every possible protocol in advance.**
13. **The initial CIM is grounded in documented published interaction
    contracts, beginning with OpenAI and Anthropic.**
14. **Provider/model deviations do not redefine CIM semantics; they belong to
    specialised representation Adapters.**
15. **Moderari owns the Adapter Registry and resolves Adapters
    deterministically: exact provider/model → provider/derived-family →
    same-representation No Adapter → required generic representation
    translation → none.**
16. **Adapter metadata does not require a separate family field; family is
    derived from the actual provider model identifier before the first `:`.**
17. **A generic representation Adapter must not be invoked merely to round-trip
    an interaction through CIM when client and provider representations already
    match.**

------------------------------------------------------------------------

# 19. Initial Requirements

## CIM-01 --- Adapter-Only Activation

Moderari must not introduce CIM normalization when Adapter processing is
Disabled.

## CIM-02 --- Explicit Schema

The CIM must have an explicit machine-readable schema.

## CIM-03 --- Schema Version

Every CIM schema must have an identifiable version.

## CIM-04 --- Semantic Concepts

The schema must describe semantic interaction concepts independently of
external protocol field names.

## CIM-05 --- Adapter Compatibility

An Adapter must identify the CIM schema version or versions it supports.

## CIM-06 --- External-to-Canonical Mapping

An Adapter must map its supported external representation to CIM
semantics where normalization is required.

## CIM-07 --- Canonical-to-External Mapping

An Adapter must materialize supported CIM semantics into its external
representation where required.

## CIM-08 --- No Pairwise Requirement

The architecture must not require direct X-to-Y Adapter implementations
for every supported representation pair.

## CIM-09 --- Published-Semantic Preservation

A base representation Adapter/CIM translation must not silently discard
documented semantics from a supported published interaction representation.

## CIM-10 --- Published-Representation Loss Visibility

Where a conforming target published representation cannot preserve documented
source semantics, the incompatibility or loss must be explicit. Loss introduced
inside a provider/model-specific specialised Adapter is owned by that Adapter
and is not a CIM schema responsibility.

## CIM-11 --- Representation Recognition Boundary

The CIM must not identify external representations. Moderari's Adapter
Framework determines client representation from defined request-boundary
evidence and installed representation contracts; Praebere supplies provider
representation. Ambiguous client representation must remain Unknown rather
than be guessed.

## CIM-12 --- Filter Independence

Moderari Filters must not depend on the CIM schema or external protocol
representations.

## CIM-13 --- Trace Independence

CIM runtime instances must not replace or redefine the Vestigare raw
conversation trace.

## CIM-14 --- Evolvability

The CIM schema must be capable of controlled evolution without requiring
a new pairwise translation architecture.

## CIM-15 --- Published Contract Basis

The initial CIM semantic model must be derived from documented external
interaction contracts, beginning with the published OpenAI and Anthropic
schemas, rather than from observed provider/model quirks.

## CIM-16 --- Quirk Exclusion

Provider/model-specific deviations from a declared representation must not be
added to the CIM merely to accommodate that implementation. They belong to a
specialised Adapter for the relevant base representation.

## CIM-17 --- Specialised-Adapter Loss Boundary

The CIM is not responsible for preserving non-standard provider/model
representations introduced by a specialised Adapter. The specialised Adapter
author owns the mapping back to the base published representation, including
any approximation or information loss introduced by that override.

## CIM-18 --- Registry Independence

Adapter discovery, registration and matching are Moderari Adapter Framework
responsibilities. They must not become CIM schema semantics.

## CIM-19 --- Deterministic Effective Adapter

Provider-side specialised Adapter resolution must yield at most one effective
specialisation. Resolution uses exact provider/model, provider/derived-family,
then a same-representation pass-through test before any generic representation
translation is considered. Exact-model and family specialisations must not be
chained.

## CIM-20 --- Recognition Independence

Representation recognition must remain outside the CIM schema. Adding a new
recognisable external representation should normally require an Adapter
extension, not a CIM protocol-detection change.

## CIM-21 --- Same-Representation Short Circuit

If client and provider interaction representations are the same and no
specialised provider/model or provider/family Adapter applies, Moderari must
use the No Adapter path. CIM normalization must not be introduced solely
because a generic Adapter for that representation is installed.

------------------------------------------------------------------------

# 20. Deferred Work

This document deliberately does not define:

``` text
the final CIM 1.0 schema
the schema technology
the exact opaque-extension mechanism
the exact loss-handling policy
streaming canonicalization
the first complete protocol Adapter
Compaction/Cognition Filter behaviour
future client-owned context behaviour
```

Those decisions should be made during review and implementation work
against concrete Adapter requirements.

The immediate architectural objective is to establish the role and
boundaries of the CIM before those implementation choices are made.


# 22. Client-Representation Materialisation

The CIM may be used for a second Adapter-Framework purpose in addition to cross-representation provider translation: constructing interaction material in the session's recognised client representation for a Moderari Filter.

``` text
Filter semantic requirement
        │
        ▼
CIM runtime instance
        │
        ▼
client representation Adapter
        │
        ▼
client-representation material
```

This does not make CIM the Filter working model. The CIM runtime instance is transient; the materialised result belongs to the client-representation working domain.

The same published-contract mapping used by an Adapter for interoperability is therefore reused for construction. Lumen must not create a second protocol-template or representation-definition system merely for Filters.

## CIM-22 — Loopback Materialisation

CIM semantics may be materialised through the Adapter for the recognised client representation when a Filter needs to create valid client-representation interaction material.

## CIM-23 — Filter Working-Domain Separation

A Filter does not retain or manipulate CIM as its working context. CIM remains a transient Adapter-Framework semantic model; the Filter receives/works with the materialised client representation.

## CIM-24 — Vestigare Boundary

CIM/provider-side intermediate representations are not Vestigare conversation records. The recorded response is the response actually returned to the client after any required Adapter translation into the client interaction representation.
