# Lumen Moderari Adapter Architecture

## Protocol and Representation Compatibility Extensions

**Date:** 02026-10-01\
**Status:** Working architecture for the current Moderari redesign

# 1. Purpose

Moderari Adapters provide technical compatibility between the
representations used at the client boundary and the provider/model
execution boundary.

Adapters are not user-selected model behaviours and they are not
intentional prompt transformations. They exist so that Lumen can connect
clients, providers and models that do not naturally express an
interaction in the same representation (protocol).

The current Qwen tool-call translator is the first Specific Adapter
implementation. The architecture must, however, be broad enough to
support protocol and representation translation without creating a
separate pairwise Adapter for every possible combination.

The governing principle is:

> **When Adapter processing is Enabled, a Moderari Adapter translates
> between an external interaction representation and Moderari's internal
> Canonical Interaction Model (CIM).**

When Adapter processing is Disabled, Moderari does not introduce CIM
normalization. The CIM is part of the Adapter Framework, not the
universal internal representation of vanilla Moderari.

# 2. Do Not Create a Lumen Protocol

Lumen should not create another network protocol merely to bridge
existing AI protocols.

Instead, Moderari uses an internal **Canonical Interaction Model**
(CIM). The CIM is an in-process semantic data model. It is not a public
wire protocol and nothing is required to speak "Lumen protocol" over the
network.

``` text
External Representation
        │
        │ normalize
        ▼
Canonical Interaction Model
        │
        │ materialize
        ▼
External Representation
```

The CIM exists so that enabled Adapters do not need pairwise knowledge
of every external protocol.

Filters do not use CIM as their working representation. When a Filter creates interaction material, it produces that material in one external interaction representation supported by Lumen. If that representation differs from the recognised client interaction representation, the Adapter Framework's loopback facility translates the Filter-produced material into the client representation before it is incorporated into the session context. The Filter does not construct or consume CIM; CIM remains internal to the Adapter Framework.

# 3. Avoid Pairwise X-to-Y Adapters

Without an internal canonical representation, supporting N external
representations tends toward pairwise translators:

``` text
OpenAI    -> Anthropic
OpenAI    -> Meta
Anthropic -> OpenAI
Anthropic -> Meta
Meta      -> OpenAI
Meta      -> Anthropic
...
```

That does not scale.

With the CIM:

``` text
OpenAI    <-> CIM
Anthropic <-> CIM
Meta      <-> CIM
Gemini    <-> CIM
Other     <-> CIM
```

Adding another external representation requires support between that
representation and the CIM, not translators to every already-supported
representation.

Conceptually:

``` text
OpenAI client
    │
    ▼
OpenAI Adapter
    │
    ▼
Canonical Interaction Model
    │
    ▼
Anthropic Adapter
    │
    ▼
Anthropic provider
```

The reverse path uses the same architectural principle.

# 4. Canonical Interaction Model

The Canonical Interaction Model (CIM) is part of the Moderari Adapter
Framework.

It is used when execution-path Adapter processing is Enabled and a resolved Adapter path requires canonical translation, or whenever the Filter Framework requires loopback translation from a Filter's output representation into the recognised client representation. Filter loopback is independent of the execution-path Adapter Enabled/Disabled setting.

``` text
ADAPTERS ENABLED

External representation
       │
       ▼
Adapter
       │
       ▼
Canonical Interaction Model
       │
       ▼
Adapter
       │
       ▼
External representation
```

When Adapter processing is Disabled:

``` text
ADAPTERS DISABLED

Client representation
       │
       ▼
Moderari
       │
       ▼
Provider representation
```

Moderari does not normalize the interaction into the CIM merely because
the CIM exists.

> **No required Adapter translation or client-representation materialisation means no CIM runtime path.**

The CIM is an internal semantic mapping model, not a network protocol
and not the normal data model for Moderari Filters.

Different external representations can map equivalent semantic concepts
onto the same CIM object. For example:

``` text
OpenAI                    CIM                    Anthropic
tool_calls[]   ───────►   tool_call   ◄───────   tool_use
function.name  ───────►   tool.name   ◄───────   name
arguments      ───────►   arguments   ◄───────   input
```

The CIM requires an explicit, versioned and evolvable schema. The
detailed schema architecture is defined separately in:

``` text
Lumen_Moderari_Canonical_Interaction_Model_Architecture
```

That architecture covers semantic objects, schema versioning,
Adapter-to-CIM compatibility, information preservation and the handling
of potentially lossy mappings.

This Adapter document deliberately does not duplicate the CIM schema
design.

The important constraints here are:

1.  the CIM is internal to the Moderari Adapter Framework;
2.  the CIM is used only for Adapter translation or client-representation materialisation;
3.  Filters do not use the CIM as their working context representation; they produce interaction material in a supported external representation and the Adapter Framework translates that material into the recognised client representation when required;
4.  Adapters normalize external representations into the CIM and
    materialize CIM semantics into external representations;
5.  Adapter translation must preserve the semantic interaction rather
    than intentionally alter it; and
6.  unsupported semantic information must not be silently discarded.

# 5. Representation Authority and Client Representation Recognition

Moderari should not require Pontis to understand AI interaction
representations merely because Pontis transports the request.

## 5.1 Client Side --- Moderari Adapter Framework

Pontis owns client request/response transport and session routing. It forwards
the ordinary interaction and `session_id` to Moderari without classifying the
AI interaction representation.

Moderari's Adapter Framework **attempts to recognise** the client interaction
representation from the request boundary using the representation definitions
supplied by installed representation Adapters.

Recognition may use authoritative request-boundary evidence such as:

``` text
request endpoint/path
headers or content type where relevant
published request structure/schema
other representation-specific boundary facts declared by the Adapter
```

This is controlled representation recognition, not arbitrary protocol guessing. **Lumen-supplied Base Representation Adapters** provide the recognition, validation and CIM mapping for the published external representations supported by Lumen.

**Base Representation Adapters are part of Lumen's CIM compatibility boundary and are not third-party extensions.** Adding support for a new external interaction representation may require corresponding changes to the CIM and is therefore a Lumen architecture/versioning change.

Third-party Adapter extensions are **Specialised Adapters**. They specialise an existing supported Base Representation Adapter for known provider/model deviations without defining a new external representation or extending the CIM.

Recognition is best-effort knowledge used for Adapter resolution. It is not a
gate on execution.

``` text
Client
  │
  ▼
Pontis
  │ session/transport routing only
  ▼
Moderari Adapter Framework
  │
  ├── Representation Resolver
  │      │
  │      ├── recognised representation
  │      │
  │      └── Unknown
  │
  └── Adapter Resolver
          │
          ├── applicable Adapter positively resolved
          │       └── use Adapter
          │
          └── no applicable Adapter positively resolved
                  └── No Adapter / pass-through
                           │
                           ▼
                        Provider
```

If more than one representation remains plausible, or no installed
representation definition can positively establish the representation,
Moderari records the client representation as `Unknown`.

`Unknown` describes Moderari's knowledge of the client representation. It is
not inherently an execution failure.

Moderari must not guess the representation or invent a translation. When the
session has **No Filter Profile**, an `Unknown` client representation selects
the **No Adapter** path and Moderari passes the interaction to the provider
unchanged. When Filters are selected, however, Moderari must positively know
the client representation in order to materialise Final Filter Output safely;
an `Unknown` client representation therefore causes Moderari to reject the
first Ask before Filter execution.

The provider may support the interaction representation even though Moderari
does not recognise it. If it does, execution proceeds normally. If it does
not, the provider's resulting error is returned through the normal interaction
path.

> **Unknown describes what Moderari knows. No Adapter describes what Moderari
> does when it cannot positively establish a compatibility transformation.**

> **Pontis transports the interaction. Moderari attempts to recognise the
> client interaction representation for Adapter resolution; failure to
> recognise it does not prevent execution.**

>**Third-party Adapters may specialise representations Lumen already supports; they may not introduce new Base Representation Adapters or extend the CIM.**


## 5.2 Provider Side --- Praebere

Praebere owns provider/model knowledge and the session-to-provider/model
reservation. The provider, rather than the model itself, normally owns the
wire/API representation.

``` yaml
provider:
  id: ollama
  interaction_representation: openai-compatible

model:
  id: qwen2.5-coder:14b
```

The provider interaction representation is **Praebere provider knowledge**. It
comes from the configured/known provider interface and must not be assumed to
be discoverable from the provider's model-discovery API.

``` text
Who transports the client request?
    Pontis

Who determines the client interaction representation?
    Moderari Adapter Framework

Who knows the provider representation?
    Praebere

Who knows model characteristics?
    Praebere

Who resolves and executes compatibility transformations?
    Moderari Adapter Framework / Adapters

Who defines common canonical interaction semantics?
    CIM
```

Representation recognition belongs to the Adapter Framework. The CIM does not
detect external representations; it defines the canonical semantics to which a
recognised representation may be mapped.

# 6. Adapter Registry and Resolution

Praebere supplies the provider interaction representation as part of the
authoritative reserved-session execution information during configuration.
Adapter resolution therefore treats provider representation as configuration
input rather than a first-Ask discovery problem. If required provider
representation knowledge is absent, the session configuration is incomplete;
Moderari must not invent it or reinterpret that absence as a pass-through
decision.


Adapter implementations discovered by Moderari form an internal **Adapter
Registry**.

The registry is Moderari knowledge. Praebere does not store or return Adapter
identities.

``` text
Praebere
  │ provider/model/provider-representation facts
  ▼
Moderari Adapter Resolver
  │
  ├── Representation Resolver result
  │     └── client representation / Unknown
  │
  └── Adapter Registry
        ├── Lumen Base Representation Adapters
        └── Specialised provider/model Adapters
  │
  ▼
effective execution path
```

Adapter resolution uses representation knowledge on both sides of the
client↔provider boundary, but the two sides are established differently.

The client interaction representation is a first-Ask request-boundary fact.
Moderari's Representation Resolver attempts to recognise it and may return a
supported representation or `Unknown`.

The provider interaction representation is authoritative Praebere knowledge
supplied during session configuration with the reserved provider/model facts.
It is therefore a configuration precondition for Adapter resolution, not a
first-Ask discovery branch. Moderari must not guess a missing provider
representation.

A Specialised Adapter does not define an independent representation or CIM
mapping. It specialises an existing Lumen Base Representation Adapter and
therefore depends upon that Base Adapter's representation↔CIM contract.

For Vanilla Moderari with No Filter Profile, an `Unknown` client representation
uses the No Adapter/pass-through path and the provider determines whether it can
process the unchanged interaction. If Filters are selected, an `Unknown` client
representation is not executable because Moderari cannot safely materialise
Final Filter Output in the client representation; the first Ask is rejected.

> **A Specialised Adapter cannot make an Unknown client representation known.
> Missing provider-representation knowledge is an incomplete session
> configuration, not a pass-through decision.**

When both client and provider representations are known, Moderari must resolve
a compatible execution path. If no compatible Base/Specialised Adapter path
exists and the representations differ, Moderari rejects the Ask rather than
knowingly forwarding an incompatible representation.

Only after both client and provider interaction representations are positively
known does Moderari evaluate Adapter compatibility and Specialised Adapter
applicability.

A Specialised Adapter uses the existing `provider` and `model` matching
metadata. No separate `family`, `scope`, `specificity` or priority field is
required.

The `model` value may identify either:

``` text
an exact provider model identifier
    qwen2.5-coder:14b

or the family portion
    qwen2.5-coder
```

For Adapter resolution, Moderari derives the family from the actual provider
model identifier as the portion before the first `:`.

``` text
qwen2.5-coder:14b
        │
        ├── exact model = qwen2.5-coder:14b
        └── family      = qwen2.5-coder
```

If no `:` exists, the complete model identifier is also the family value.

Resolution is deterministic:

``` text
1. client representation known?
        │
       NO ── No Filters ───────────► No Adapter / pass-through
        │
        └── Filters selected ──────► Reject first Ask
        │
       YES
        ▼
2. provider representation known?
        │
       NO ─────────────────────────► No Adapter
        │
       YES
        ▼
3. provider + exact model Specialised Adapter?
        │
       YES ────────────────────────► use Specialised Adapter
        │
        NO
        ▼
4. provider + derived model family Specialised Adapter?
        │
       YES ────────────────────────► use Specialised Adapter
        │
        NO
        ▼
5. client representation == provider representation?
        │
       YES ────────────────────────► No Adapter
        │
        NO
        ▼
6. required Lumen Base Representation Adapter path available?
        │
       YES ────────────────────────► translate through CIM
        │
        NO
        ▼
7. Reject — known incompatible representation path
```

The representation-knowledge checks deliberately precede Specialised Adapter
matching. A Specialised Adapter can override known provider/model deviations
from an existing supported Base Representation Adapter, but it cannot define
the representation required to reach the CIM.

The same-representation test is an intentional short-circuit. If client and
provider already use the same known interaction representation, and no
Specialised provider/model or provider/family Adapter applies, Moderari must
not normalize the interaction through the CIM merely because a Base
Representation Adapter is installed.

A Specialised Adapter is checked before same-representation pass-through only
after both representations are known, because it exists to handle a known
provider/model deviation from a known published representation.

For example:

``` text
client representation   = openai-compatible
provider                 = ollama
provider representation = openai-compatible
model                    = qwen2.5-coder:14b

lookup:
    (ollama, qwen2.5-coder:14b)
        ↓ none

    (ollama, qwen2.5-coder)
        ↓ match

    use that Specialised Adapter
```

By contrast:

``` text
client representation   = openai-compatible
provider                 = experimental-provider
provider representation = Unknown
model                    = qwen2.5-coder:14b

result:
    provider representation Unknown
        ↓
    No Adapter
        ↓
    pass interaction unchanged
```

Even if a matching Qwen Specialised Adapter is installed, it cannot be selected
for this execution because the Base Representation Adapter/CIM contract that it
specialises has not been positively established for the provider boundary.

**Only one installed Adapter implementation may declare a particular
`provider + model` registry key. Adapter version is not part of the registry
key and does not permit multiple versions of an Adapter for the same key to be
installed simultaneously.**

This uniqueness rule applies equally when the declared `model` value is a
family lookup key.

If two or more installed Adapters declare the same `provider + model` key —
**including different versions of the same Adapter** — Adapter Registry
construction fails and Moderari enters a failed startup state.

Moderari must not select a version, prefer the newest version, rank the
implementations, use discovery order, or otherwise resolve the conflict
automatically.

The startup failure must identify the conflicting registry key and each
conflicting Adapter's identity and version so that Servire and the operator can
diagnose the invalid configuration.

Adapter version is execution identity/provenance; it is not part of the
applicability key and Moderari does not retain historical Adapter versions
merely to make old executions replayable. Updating an Adapter therefore
replaces the previously installed implementation for normal execution.

If Repetere later requests an Original Setup whose recorded Adapter
implementation/version is no longer available, Moderari rejects reconstruction
of that Original Setup. It must not silently substitute the currently installed
version. The user may instead create a New Setup, which is resolved normally
against the currently installed Adapter set.

Moderari detects and reports the invalid Adapter configuration. Servire owns
the resulting Stack recovery/rollback behaviour.

An exact-model match takes precedence over a family match. Exact-model and
family specialisations are not chained. How an exact-model implementation
reuses code from a family specialisation, if desired, remains an extension
implementation concern.

Moderari selects one effective Specialised Adapter for the provider side when
the representation prerequisites are known and an exact-model or family match
exists. Otherwise it follows the same-representation, Base Representation
Adapter translation, or No Adapter path described above.

> **Unknown describes what Moderari knows. No Adapter describes what Moderari
> does when it cannot positively establish a compatibility transformation.**

> **Unknown on either side of the interaction boundary means Moderari has
> insufficient representation knowledge to translate. Insufficient knowledge
> results in No Adapter, not guessed translation or execution rejection.**

> **Unknown does not mean broken. Unknown means don't interfere.**

# 7. Protocol Adapters and Provider/Model-Specific Specialisation

The base Adapter for an external interaction representation implements that
representation according to its published contract.

Examples include:

``` text
OpenAI Adapter
    published OpenAI representation <-> CIM

Anthropic Adapter
    published Anthropic representation <-> CIM
```

The CIM and base protocol/representation Adapters must remain faithful to
published representation semantics. Provider/model quirks must not be folded
into the CIM merely because a particular implementation deviates from the
published contract.

Where a provider/model combination requires special handling, a specialised
Adapter may derive from the relevant protocol/representation Adapter and
supply only the required differences.

Conceptually:

``` text
OpenAI Adapter
    │
    │ complete published OpenAI mapping
    │
    └── Qwen/OpenAI Specialisation
            only provider/model-specific overrides
```

The specialisation inherits/reuses the standard representation behaviour.
Anything not explicitly overridden retains the behaviour of the base
representation Adapter.

This is **specialisation, not an Adapter correction chain**. Moderari should
not need to execute:

``` text
OpenAI Adapter
    -> Qwen Correction Adapter
    -> another correction Adapter
```

Instead, the effective Qwen/OpenAI Adapter is the OpenAI representation
Adapter plus the explicitly supplied Qwen/provider-specific overrides.

The exact mechanism by which this is implemented is deliberately open. It may
eventually use class inheritance, delegation, composition, declared override
hooks or another extension mechanism. The architecture requires only that a
specialised Adapter provide the differences rather than reimplement the
published representation mapping.

Lumen Core is not responsible for discovering or inventing undocumented
provider/model corrections. If a provider/model deviates from its declared
representation and no applicable specialised Adapter exists, Lumen remains
faithful to the declared/published representation contract and preserves what
the provider/model actually returns.

Where a specialised Adapter deliberately converts a provider/model-specific
deviation into the base published representation, the specialised Adapter
author owns that conversion. Any information loss, approximation or semantic
compromise introduced by that override is therefore an Adapter implementation
issue, not a reason to broaden the CIM or alter the published base Adapter.

This is distinct from a genuine interoperability loss between two conforming
published representations. If a semantic concept defined by one supported
published representation cannot be represented by another, that remains an
Adapter/CIM interoperability condition and must not be silently hidden.

The current Qwen tool-call translator is the first concrete provider/model
specialisation case. OpenAI and Anthropic protocol/representation Adapters are
the initial published contracts against which the CIM should be developed.
Implementation of the full set remains a separate implementation decision.

# 8. Adapter Discovery and Plugin Packaging

Moderari Core owns the Adapter Framework.

Base Representation Adapters are Lumen-supplied components of the versioned
CIM compatibility boundary. They are not third-party extensions. Adding a new
Base Representation Adapter is a Lumen architecture/versioning change because
the new representation may require corresponding CIM semantics.

Independently installable third-party Adapter extensions are **Specialised
Adapters**. They may specialise an existing Lumen-supported Base
Representation Adapter for provider/model-specific deviations, but they may
not introduce a new Base Representation Adapter or extend the CIM.

The Lumen Research Foundation remains a single Docker image/container
distribution. External Adapter code becomes visible inside the running
container through a Docker bind mount or volume.

Conceptually:

``` text
Host
/lumen-extensions/adapters
        │
        │ Docker bind mount
        ▼
Lumen Research Foundation container
/lumen/extensions/adapters
        │
        ▼
Moderari Adapter Discovery
```

A third-party Specialised Adapter does not need to be built into the Lumen
image. Base Representation Adapters remain Lumen-supplied components of the
versioned CIM compatibility boundary.

A package may initially resemble:

``` text
my-adapter/
├── lumen-extension.yaml
├── adapter.py
├── README.md
└── LICENSE
```

The manifest is inspected before executable code is loaded.

A conceptual manifest:

``` yaml
extension:
  type: adapter
  id: qwen-tool-call
  version: 1.0.0
  lumen_extension_api: "1"

entrypoint:
  module: adapter
  class: QwenToolCallAdapter

matching:
  provider: ollama
  model: qwen2.5-coder
  representation: openai-compatible
```

Third-party packages do not declare new Base Representation Adapters.
A Specialised Adapter declares the existing Lumen-supported Base
Representation Adapter/representation that it specialises, together with its
provider/model matching criteria.

# 9. Adapter Plugin Contract

An Adapter should be able to self-describe:

``` text
identity
implementation version
extension API version
description
entry point
supported direction(s)
existing Lumen-supported base representation being specialised
provider/model matching criteria
required characteristics where genuinely needed
configuration schema, if any
```

Moderari Core must not contain:

``` python
if model_is_qwen:
    ...
elif provider_is_anthropic:
    ...
```

Instead, installed Adapters declare their applicability and Moderari
evaluates those declarations against authoritative session facts.

Praebere must never know installed Adapter identities.

# 10. Adapter State and User Control

Users do not select individual Adapters.

The user-facing control applies to the normal client↔provider execution path:

``` text
Execution Adapter processing: Enabled / Disabled
```

When enabled, Moderari automatically resolves and applies the required
effective installed Adapter implementation for each external representation.
On the provider side this follows the deterministic exact-model → family →
same-representation pass-through → required generic representation translation
→ none resolution order.

When disabled, Moderari does not apply Adapter transformations on the normal client↔provider execution path and does not introduce CIM normalization merely for provider execution.

This setting does **not** disable Filter loopback translation. Where a Filter's output representation differs from the recognised client interaction representation, loopback translation is mandatory so that the Filter insertion is valid in the client's representation.

The operational state remains deliberately simple:

``` text
Enabled
Disabled
```

`Disabled` does not assert why no Adapter is being applied.

A conceptual internal Pass Through Adapter may represent the
no-transformation path.

# 11. Dependencies and Trust

A third-party Adapter executes code inside the Moderari process when
loaded in-process.

Therefore installation is a trust decision.

Initial Research Foundation behaviour should be conservative:

``` text
Extension discovered
    │
    ▼
Manifest inspected
    │
    ▼
User/administrator explicitly enables/trusts extension
    │
    ▼
Moderari loads executable code
```

The initial extension API should prefer dependencies already present in
the Research Foundation runtime. Arbitrary runtime `pip install` into
the Lumen container should not be part of the first implementation.

Heavyweight or isolated extension runtimes can be considered later if a
real requirement emerges.

# 12. Development Workflow

A researcher should be able to develop an Adapter outside the Lumen
source tree:

``` text
C:\Development\My-Lumen-Adapter
        │
        │ bind mount
        ▼
/lumen/extensions/adapters/my-adapter
        │
        ▼
Moderari discovery
```

The Lumen Research Foundation image remains unchanged.

For the first implementation, restart-based discovery is sufficient. Hot
unloading/reloading of arbitrary Python extension code is outside the
initial scope.

# 13. Adapters and the Raw Trace

Adapters are not semantic interaction transformations.

They translate representation for compatibility while preserving the
information being represented.

``` text
Representation A
      │
      ▼
Canonical Interaction Model
      │
      ▼
Representation B
```

The representation may change; the interaction does not.

Vestigare therefore records the true client/model conversation
independently of the intermediate Adapter representation used inside
Lumen.

Adapter identity/version may be useful as Moderari operational/session
metadata, but Adapter representation changes are not alternate
conversation content and are not Filter transformation evidence.

This is the fundamental distinction:

> **Adapters change presentation/representation. Filters intentionally
> change the interaction presented to the model.**

------------------------------------------------------------------------

# 14. Current Implementation Scope

The current redesign supplies:

``` text
Adapter Framework
│
├── Base Representation Adapters
│     ├── OpenAI Adapter
│     └── Anthropic Adapter
│
└── Specialised Adapters
      └── Qwen Adapter
```

OpenAI and Anthropic implement their published external interaction representations against CIM. Qwen is supplied as a specialised Adapter for known Qwen deviations from its applicable base representation.

The framework must remain extensible and must not make OpenAI, Anthropic or Qwen architectural special cases.

# 15. Adapter Requirements

## MAR-01 --- Discoverable Extensions

A compliant Adapter can be installed outside Moderari Core and
discovered through the configured extension location.

## MAR-02 --- Canonical Translation

When execution-path Adapter processing is Enabled, Adapters translate between an
external representation and the Moderari Canonical Interaction Model
rather than requiring pairwise X-to-Y translators. When execution-path Adapter
processing is Disabled, the CIM is not introduced merely for client↔provider execution. This setting does not disable mandatory Filter loopback translation.

## MAR-03 --- Dual-Sided Representation Resolution

Adapter resolution uses the client-representation recognition result from
Moderari's Adapter Framework together with the provider interaction
representation and provider/model information supplied by Praebere during
session configuration. The client representation may be `Unknown`; the provider
representation is required configuration input. Generic or Specialised Adapter
translation requires a recognised client representation and the configured
provider representation.

## MAR-04 --- Controlled Client Representation Recognition

Moderari's Adapter Framework must attempt to recognise the client interaction
representation from the request boundary using installed representation
definitions and published-contract recognition/validation rules. When the
available evidence is ambiguous or insufficient, it must record the
representation as `Unknown` rather than guess. `Unknown` must not itself prevent
execution.

## MAR-05 --- Praebere Independence

Praebere knows provider/model facts but has no knowledge of installed
Moderari Adapters.

## MAR-06 --- Automatic Selection

Users never select an Adapter implementation. Moderari resolves
applicable Adapters automatically.

## MAR-07 --- Enable/Disable Control

The user may enable or disable Adapter processing on the normal client↔provider execution path for the session. This control must not disable Filter loopback translation.

## MAR-08 --- Pass Through and Rejection

On the normal client↔provider execution path, an `Unknown` client representation
with **No Filter Profile** uses the No Adapter/no-transformation path. The
interaction is passed to the provider unchanged and the provider may accept it
or return its normal error.

If Filters are selected, an `Unknown` client representation must cause Moderari
to reject the first Ask because Final Filter Output cannot be safely materialised
in an unrecognised client representation.

When client and provider representations are both known but differ, and no
compatible Adapter path exists, Moderari must reject the Ask rather than knowingly
forward an incompatible representation. Execution Adapter processing being
disabled remains an explicit user-selected no-transformation path; it does not
disable mandatory Filter loopback translation.

## MAR-09 --- Self Description

Each Adapter declares identity, version, supported
representation/direction, matching criteria and requirements through a
machine-readable contract.

## MAR-10 --- Open Specialised-Adapter Extension Boundary

A new compliant third-party Specialised Adapter must not require
provider/model-specific changes to Moderari Core, Pontis, Rogare, Vestigare or
Repetere. Third-party Adapters may specialise an existing Lumen-supported Base
Representation Adapter; they may not introduce a new Base Representation
Adapter or extend the CIM.

## MAR-11 --- Execution Metadata

Adapter identity/version and effective application state may be recorded
as Vestigare execution metadata describing the environment under which
the raw trace occurred. Adapter representation changes do not become
alternate conversation content in the Vestigare raw trace.


## MAR-12 --- Published Contract Authority

A base protocol/representation Adapter must implement the documented published
contract for the representation it supports. Provider/model-specific quirks
must not silently redefine that base contract.

## MAR-13 --- Specialised Adapter Overrides

A provider/model-specific Adapter must specialise an applicable base
protocol/representation Adapter and provide only the deviations required for
that provider/model combination. Behaviour not explicitly overridden must
retain the base Adapter semantics.

## MAR-14 --- Specialisation Mechanism Deferred

The mechanism used to realise Adapter specialisation is an implementation
design decision. The architecture must not require a particular language-level
inheritance mechanism.

## MAR-15 --- No Silent Compensation

Moderari Core must not infer or silently compensate for undocumented
provider/model deviations. Without an applicable specialised Adapter, Lumen
uses the declared representation contract and preserves the actual returned
interaction as evidence.

## MAR-16 --- Specialisation Owns Override Loss

Where a specialised provider/model Adapter overrides the behaviour of a base
published-contract Adapter, the specialised Adapter owns the semantics and any
lossiness of that override. Such loss must not cause provider/model quirks to
be incorporated into the CIM.


## MAR-17 --- Adapter Registry

Moderari must build and own a registry of discovered Adapter extensions.
Praebere must not maintain or return installed Adapter identities.

## MAR-18 --- Deterministic Dual-Sided Resolution

For a provider/model execution target, Moderari must resolve the effective
execution path in this order:

``` text
client representation Unknown + No Filters -> No Adapter / pass-through
client representation Unknown + Filters selected -> reject first Ask
provider + exact model Specialised Adapter
provider + derived model family Specialised Adapter
same known client/provider representation -> No Adapter
required Lumen Base Representation Adapter path
No Adapter
```

Specialised Adapter matching occurs only after both client and provider
interaction representations are positively known because a Specialised Adapter
depends upon an existing Base Representation Adapter/CIM mapping.

The model family is derived as the portion of the provider model identifier
before the first `:`. No separate family metadata is required in the Adapter
manifest.

## MAR-19 --- Registry Key Uniqueness and Startup Failure

At most one installed specialised Adapter may declare any particular
`provider + model` registry key.

If duplicate keys are discovered during Adapter Registry construction,
Moderari must fail registry construction and enter a failed startup state. It
must not resolve the conflict through discovery order, version preference,
ranking, scoring, automatic disabling or priority.

The failure information must identify the duplicate key and the conflicting
Adapter identities/versions. Servire is responsible for treating the failed
Moderari startup as a Stack startup failure and invoking the normal Stack
rollback mechanism.

## MAR-20 --- Single Effective Specialisation

An exact-model specialisation replaces a family specialisation for that
resolution. Moderari must not execute exact-model and family specialisations as
a correction chain.

## MAR-21 --- Same-Representation Pass Through

When client and provider interaction representations are the same, and no
specialised provider/model or provider/family Adapter applies, Moderari must
select the No Adapter path. It must not perform a generic representation ->
CIM -> same representation round trip merely because a generic Adapter exists.

A specialised Adapter takes precedence over this short-circuit because it
represents a known provider/model deviation from the nominal representation.

## MAR-22 --- Provider Representation Knowledge

Provider interaction representation is Praebere provider knowledge derived
from the configured/known provider interface, or from explicit provider
metadata where available. Lumen must not assume that model-discovery APIs
report the provider interaction representation.

## MAR-23 --- Pontis Representation Independence

Pontis must not be required to classify the client's AI interaction
representation. It forwards the interaction and session identity to Moderari
as part of its transport/session-routing responsibility.

## MAR-24 --- Representation Recognition Is Not CIM

Client representation recognition belongs to the Moderari Adapter Framework.
The CIM schema must not contain protocol-detection or representation-recognition
logic.

## MAR-24A --- Unknown Representation Pass Through

An `Unknown` client interaction representation is not inherently an execution failure. Moderari must not guess or invent a translation.

With **No Filter Profile**, an `Unknown` client representation selects the No Adapter path and the unchanged interaction may be passed to the provider.

With selected Filters, an `Unknown` client representation must cause Moderari to reject the first Ask because the Filter Framework cannot safely produce Final Filter Output in the required client representation.

When both client and provider representations are positively known but differ, Moderari must establish a compatible Adapter path. If no such path exists, the Ask is rejected as a known incompatibility.

A Specialised Adapter cannot make an Unknown representation known. It depends on the existing Base Representation Adapter/CIM mapping for the representation it specialises.

The first-Ask decision is therefore:

``` text
Client Unknown + No Filters                  -> pass through
Client Unknown + Filters selected            -> reject
Both known + no compatible Adapter path      -> reject
Both known + compatible execution path       -> execute
```

# 16. Acceptance Criterion

> **A new Moderari Adapter should be independently installable and
> discoverable; translate between an external interaction representation
> and Moderari's Canonical Interaction Model; declare its own matching
> criteria; be registered by Moderari and resolved deterministically using
> exact provider/model, provider/derived-family, same-representation
> pass-through, required generic representation translation, then no Adapter; use the client
> representation recognition result from Moderari, including `Unknown`, together
> with provider/model/representation facts supplied by Praebere; treat `Unknown`
> as pass-through rather than an execution failure;
> support deliberate Adapter disable/pass-through behaviour; and expose
> generic provenance without requiring Adapter-specific changes to
> Moderari Core, Rogare, Vestigare or Repetere.**


# 25. Client-Representation Loopback Facility

A Filter intentionally inserts or changes interaction material presented to the model. That material must ultimately be incorporated into the Moderari session context in the **recognised client interaction representation**.

A Filter is not required to implement every external interaction representation supported by Lumen. Instead, a Filter may produce its insertion in one supported external representation that the Filter knows how to construct.

The Filter Framework owns the Filter execution. It creates the `filter_execution_id`, invokes the Filter, and receives the Filter-produced material. If that material is already in the recognised client representation, it directly becomes the **Final Filter Output**. If the representations differ, the Filter Framework uses the Adapter Framework in **loopback** to translate the material into the recognised client representation. The translated result is returned to the Filter Framework and becomes the Final Filter Output.

``` text
Filter Framework
   │
   │ creates filter_execution_id
   │ invokes Filter
   ▼
Filter
   │
   │ Filter-produced material
   │ in one supported external representation
   ▼
Filter Framework
   │
   ├── same as client representation ────────────┐
   │                                             │
   └── different                                 │
          │                                      │
          ▼                                      │
     Adapter Framework                           │
     Loopback Translation                        │
          │                                      │
          │ Adapter/CIM machinery                │
          ▼                                      │
     Client-Representation Material ─────────────┘
                      │
                      ▼
               Filter Framework
                      │
                      ▼
               Final Filter Output
             (client representation)
                      │
                      ├── persist as Filter
                      │   execution evidence
                      │
                      └── incorporate into
                          session context
                              │
                              ▼
                   normal client → provider
                        execution path
```

CIM remains internal to the Adapter Framework. The Filter does not construct CIM, consume CIM, or define a new Lumen-internal interaction representation. Loopback reuses the existing Adapter mappings between supported external representations and CIM to perform the required translation.

**Filter-produced material** is the material initially returned by the Filter implementation in its supported external representation. **Final Filter Output** is that material expressed in the recognised client interaction representation after any required loopback translation.

Loopback translation is part of Filter execution, but the Adapter Framework does not own the resulting Filter output. After translation, the client-representation material is returned to the Filter Framework. Moderari persists the Final Filter Output as Filter execution evidence and incorporates that same Final Filter Output into the session context. The Adapter Framework does not persist the translated result as independent Adapter output or interaction content.

For example, a System Prompt Filter may natively produce an OpenAI-form interaction insertion. With an OpenAI client, that Filter-produced material directly becomes the Final Filter Output. With an Anthropic client, loopback translates the Filter-produced OpenAI material into the Anthropic client representation; the returned Anthropic material then becomes the Final Filter Output. The complete client-representation interaction subsequently continues through the normal execution path and may be translated again if the provider representation differs.

Loopback translation is a Filter Framework construction requirement, not a user-selected execution compatibility option. The session's execution Adapter Enabled/Disabled control applies only to the normal client↔provider execution path. It **must never disable loopback translation**.

> **Where a Filter's output representation differs from the recognised client interaction representation, loopback translation is mandatory.**

> **Filter Framework owns the execution and Final Filter Output. Adapter Framework owns only the representation translation required to materialise that output.**

The loopback facility does not change Adapter resolution for provider execution. After the Final Filter Output has been incorporated into the client-representation context, normal provider execution rules apply: same-representation provider execution uses No Adapter unless a specialised Adapter applies; a different provider representation uses the required Adapter/CIM translation when execution Adapter processing is Enabled.

### MAR-25 — Filter Output Representation

A Filter may produce interaction material in one external interaction representation supported by Lumen. A Filter must not be required to implement construction logic for every interaction representation supported by Lumen. The material initially returned by the Filter is Filter-produced material; it becomes the Final Filter Output only after it is expressed in the recognised client interaction representation.

### MAR-26 — Mandatory Client-Representation Loopback

Where a Filter's output representation differs from the recognised client interaction representation, the Adapter Framework must translate that Filter-produced material into the client interaction representation and return the result to the Filter Framework. That returned client-representation material becomes the Final Filter Output before it is persisted as Filter execution evidence and incorporated into the session context. This loopback translation is mandatory and must not be disabled by the session's execution Adapter Enabled/Disabled setting. The Adapter Framework must not persist the translated result as independent Adapter output or interaction content.

### MAR-27 — No Filter CIM Dependency

A Filter must not construct or consume CIM merely to express its intentional transformation. CIM remains internal to the Adapter Framework and is used by the loopback machinery where representation translation is required.

### MAR-24B --- Base Representation Adapter Governance

Base Representation Adapters are Lumen-supplied components of the versioned
CIM compatibility boundary. Third-party Adapter extensions may specialise
existing supported Base Representation Adapters but must not introduce new Base
Representation Adapters or extend the CIM.

## MAR-24C --- Original Setup Adapter Availability

Adapter implementation/version is recorded as execution provenance but
historical Adapter versions are not retained as a replay requirement. If an
Original Setup requests an Adapter implementation/version that is no longer
available, Moderari must reject that Original Setup rather than substitute a
different version. A New Setup is resolved normally against the current
Adapter Registry.

## MAR-28 — Adapter Evidence Restriction

An Adapter extension must have no authority to add, remove, replace or suppress Vestigare raw interaction records. Vestigare trace capture is a Lumen Core responsibility at the Client ↔ Lumen boundary.
