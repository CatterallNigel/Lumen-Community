# Lumen Moderari Filter Architecture

## Intentional Interaction Transformation Extensions

**Date:** 02026-10-01\
**Status:** Working architecture for the current Moderari redesign

# 1. Purpose

Moderari Filters are explicit, user-selected transformations of an
interaction.

They are separate from Adapters:

> **Adapters provide technical compatibility. Filters provide
> intentional transformation.**

Filters operate on Moderari-managed interaction/context material in the session's recognised **client interaction representation**. They do not use the provider representation as their working domain and they do not construct or consume the CIM.

When a Filter creates new interaction material, it produces that material in one external interaction representation supported by Lumen that the Filter knows how to construct. If that representation differs from the recognised client interaction representation, the Filter Framework uses the Adapter Framework's **client-representation loopback facility** to translate the Filter-produced material into the client representation. The CIM remains internal to the Adapter Framework.

The result after any required loopback processing is the **Final Filter Output**. It is always in the recognised client interaction representation and is the Filter output that Moderari persists and incorporates into session context.

# 2. Current Scope

The current Lumen redesign supplies only two Filters:

``` text
System Prompt Filter
Cognition Filter
    working name; previously "Compaction"
```

No additional built-in Filters should be added during the current
redesign.

The extension framework must deliberately leave room for researchers to
create their own Filters later.

Illuminates.One can experiment with additional Filters after the present
architecture and implementation work is complete.

# 3. Vanilla Moderari

Zero Filters is a first-class configuration.

``` text
Client interaction
       │
       ▼
Moderari-managed session context
       │
       │ Filters: []
       ▼
Provider/model path
```

Vanilla Moderari does not invent a System Prompt.

For the current redesign Moderari continues to retain the session
conversation/model-facing context required by the existing interaction
and Filter machinery. Moving full conversational-context ownership to
clients such as Rogare is a later requirement and is outside the present
redesign.

# 4. Context Ownership --- Current Scope

For the current redesign, Moderari retains the session conversation and
model-facing context required by the existing implementation.

This is intentionally conservative while the Moderari code is reworked,
particularly because the existing Cognition behaviour must be
inspected before its replacement semantics are fixed.

``` text
Rogare / Client
      │
      │ interaction
      ▼
Moderari
      │
      ├── maintains current session context
      ├── applies selected textual/context Filters
      └── maintains any Filter-specific context state
```

A later requirement may move complete conversational-context ownership
to clients such as Rogare. That future design must define how
client-owned raw context is reconciled with Moderari-managed
transformed/model-facing context.

That work is explicitly deferred.

# 5. System Prompt Filter

Moderari has no default System Prompt.

If no System Prompt Filter is selected, Moderari does not add one.

Any Lumen/user System Prompt must therefore be represented through a
Moderari Filter Profile containing the System Prompt Filter.

**Creating a Custom System Prompt in Moderari automatically creates a Moderari Filter Profile with the same name containing the System Prompt Filter configured with that prompt.**

The Custom System Prompt and its corresponding Filter Profile therefore begin as a single user action rather than requiring the user to create and associate the two resources separately.

``` text
Filter Profile: "My Research Prompt"
│
└── System Prompt Filter
       └── prompt = <configured prompt>
```

The user may subsequently add another Filter to the same profile:

``` text
Filter Profile: "My Research Prompt"
│
├── System Prompt Filter
└── Cognition Filter
```

Saved System Prompts remain configuration-time resources. Creating a saved Custom System Prompt automatically creates its same-named Filter Profile. Prompt names must therefore be unique. Historical prompt or Filter Profile versions are not required for replay; the materialised Moderari Session Execution State records the actual effective System Prompt content used for execution.

# 6. Filter Profiles

A Moderari Filter Profile is a user convenience for selecting a reusable
Filter configuration for an existing session.

It is not an executable plugin and it is not execution provenance.

``` text
Filter Profile
      │
      ▼
resolve current definition
      │
      ├── Filter A + configuration
      └── Filter B + configuration
      │
      ▼
Effective Session Filter Set
```

The resolved Filter set is execution truth.

`No Filter Profile` remains an explicit valid selection and resolves to
zero Filters.

Filter Profile creation/editing and detailed Filter configuration belong
to the Moderari UI in Servire. Rogare selects available Filter Profiles;
it is not a Filter administration UI.

A Filter Profile remains a persistent, editable Moderari configuration resource
even if one or more Filters referenced by it are no longer available.

Profile **validity** is evaluated before model applicability:

``` text
Filter Profile
      │
      ▼
Are all referenced Filters currently available?
      │
  ┌───┴────┐
 YES       NO
  │         │
  ▼         ▼
VALID     INVALID
  │         │
  │         ├── profile still exists
  │         ├── visible/editable in Servire
  │         ├── identify missing Filter(s)
  │         └── not offered to clients
  │
  ▼
evaluate model applicability
      │
      ▼
selectable when applicable
```

An invalid profile is not deleted automatically. It remains available for
administration so that the user can edit, repair or delete it. If the missing
Filter later becomes available again, the profile may become valid again.

A profile may be offered to Rogare or another client only when it is both
**valid** (all referenced Filters are available) and **applicable** to the
selected model.

# 7. The Cognition Filter

The **Cognition Filter** captures the model's present understanding at a particular point during an interaction.

Cognition is a **snapshot of the model's deliberation given the context consumed and the aim of the user's Ask**. It is obtained by a private inference in which the model is asked for its present understanding.

Conceptually:

```
Context consumed
      +
Aim of the user's Ask
      │
      ▼
   Model
      │
      │ private Cognition Ask
      ▼
Present Understanding
      │
      ▼
Cognition Snapshot
```

The existing Moderari implementation will be inspected during implementation work to establish:

```
how session context is retained
how context consumption is measured
when the Cognition Filter is triggered
the private Ask used to obtain the model's present understanding
how the Cognition Snapshot is incorporated into subsequent model-facing context
what actual context is retained alongside it
what happens to subsequent client context and any context cached by the client
what state is retained between interactions
```

The detailed replacement-context behaviour will be fixed only after the existing implementation has been inspected.

> **Cognition is the model's present understanding at a point in time, given the context it has consumed and the aim of the user's Ask.**

**The initial Cognition Filter scope remains limited to capturing the model's present understanding for use in managing model-facing context.**

Cognition Snapshots may also provide useful evidence for assessing behavioural drift during a session. Earlier Moderari experiments periodically obtained cognition during a conversation to compare the model's present understanding with the task it was expected to perform. That mechanism was basic and is not part of the initial Cognition Filter contract, but the architecture should not prevent Cognition Snapshots from being used for this purpose in future.
## 8. Filters Produce Interaction Material in a Supported Representation

Moderari Filters intentionally add or modify material presented to the model. Filters do not construct, consume or operate on the CIM. The CIM is internal to the Adapter Framework.

A Filter produces new interaction material in **one external interaction representation supported by Lumen** that the Filter knows how to construct. This initial result is **Filter-produced material**. The Filter is not required to understand or produce every interaction representation supported by Lumen.

For example:

``` text
System Prompt Filter
    output representation: OpenAI

Cognition Filter
    output representation: OpenAI
```

The Filter-produced material must ultimately be materialised in the recognised **client interaction representation**. If the Filter output representation already matches the client representation, no translation is required. If the representations differ, the Filter Framework invokes the Adapter Framework in **loopback** to translate the Filter-produced material into the client representation.

``` text
Moderari Filter Framework
        │
        │ creates filter_execution_id
        │ invokes Filter
        ▼
      Filter
        │
        │ Filter-produced material
        │ in Filter output representation
        ▼
Moderari Filter Framework
        │
        ├── representation == client representation
        │        │
        │        └────────────────────┐
        │                             │
        └── representation differs    │
                 │                    │
                 ▼                    │
          Adapter Framework           │
              Loopback                │
                 │                    │
                 └──────────────┐     │
                                ▼     ▼
                         Final Filter Output
                      (client representation)
                                │
                                ├── persist
                                │
                                ▼
                         session context
```

The **Final Filter Output** is the result after any required loopback processing. It is always expressed in the recognised client interaction representation. Moderari persists this Final Filter Output and incorporates it into the session context. The Filter-produced pre-loopback representation and any transient CIM representation are implementation mechanics and are not separately persisted as the Filter output.

The loopback is part of Filter execution. It is not a separate persisted execution result and it is not controlled by the user-facing Enabled/Disabled setting for normal client-to-provider Adapter processing. Where loopback translation is required, it is mandatory.

The responsibilities are therefore:

``` text
Filter
    decides WHAT intentional material to add or change
    and produces it in one supported external representation

Filter Framework
    owns Filter execution
    creates filter_execution_id
    obtains the Final Filter Output
    persists it and incorporates it into session context

Adapter Framework loopback
    when required, translates Filter-produced material into
    the recognised client interaction representation

Normal Adapter path
    translates the resulting client interaction into
    the provider representation when required

CIM
    remains the Adapter Framework's internal semantic
    bridge for representation translation
```

For example:

``` text
System Prompt Filter
    produces OpenAI system-message material

Client = OpenAI
    → no loopback translation
    → Final Filter Output is the produced OpenAI material

Client = Anthropic
    → loopback OpenAI → Anthropic
    → Final Filter Output is the translated Anthropic material

Provider = OpenAI
    → normal Anthropic → OpenAI Adapter execution follows
```

> **Filters do not define a Lumen-internal interaction representation and do not use the CIM. They produce material in a supported external representation; the Filter Framework obtains the Final Filter Output in the recognised client representation, using Adapter Framework loopback when required.**

> **Moderari persists the Final Filter Output, not the transient representation machinery used to produce it.**

> **Where a Filter's output representation differs from the client interaction representation, loopback translation is mandatory. It must never be bypassed as a consequence of normal execution Adapter processing being Disabled.**

# 9. Filter Discovery and Plugin Packaging

Moderari Core owns the Filter Framework. Filter implementations are
independently installable extensions.

The Lumen Research Foundation remains a single Docker image/container
distribution. External Filter code is made visible inside the running
container through a Docker bind mount or volume.

``` text
Host
/lumen-extensions/filters
        │
        │ Docker bind mount
        ▼
Lumen Research Foundation container
/lumen/extensions/filters
        │
        ▼
Moderari Filter Discovery
```

A researcher can therefore develop a Filter outside the Lumen source
tree without rebuilding the Research Foundation image.

An initial package may resemble:

``` text
my-filter/
├── lumen-extension.yaml
├── filter.py
├── README.md
└── LICENSE
```

# 10. Self-Describing Filter Contract

A Filter must self-describe sufficiently for Moderari and the Servire
Moderari UI to discover, validate, configure and execute it.

The contract should include:

``` text
identity
implementation version
extension API version
description
entry point
execution stage
model/provider requirements
configuration schema
```

Conceptually:

``` yaml
extension:
  type: filter
  id: context-cognition
  version: 1.0.0
  lumen_extension_api: "1"

entrypoint:
  module: filter
  class: ContextCognitionFilter

stage: pre_inference

requirements:
  context:
    effective_window: known
```

The exact manifest schema remains an implementation task.

# 11. Model Capability Requirements

Filters declare requirements; they do not determine model truth.

Praebere supplies the Effective Model Definition.

Moderari evaluates Filter requirements against that definition.

``` text
Praebere
    Effective Model Definition
             │
             ▼
Moderari
    installed Filter requirements
             │
             ▼
Filter/Profile applicability
```

A positive requirement is satisfied only by an authoritative/acceptable
positive model characteristic. `unknown` does not satisfy a positive
requirement.

The model capability vocabulary must be extensible. Lumen should define
only capabilities currently required by consumers rather than attempting
to enumerate every capability any future model may possess.

Potential capability namespaces include:

``` text
interaction
    system_messages
    multi_turn

tools
    declarations
    calls
    results
    parallel_calls

output
    structured_output
    json_output

context
    maximum_window
    effective_window

generation/provider
    streaming
    representation
```

This list is not yet the final Praebere schema. It identifies the
capability families needed for Adapter/Filter applicability design.

# 12. Filter Profile Validity and Applicability

Before model applicability is evaluated, Moderari validates that every Filter
referenced by the Filter Profile is currently available. A profile containing
an unavailable Filter is **invalid for execution** and must not be exposed to
clients as selectable, although the profile itself remains stored and editable
through the Moderari UI in Servire.

Only a valid Filter Profile proceeds to model applicability evaluation.

A Filter Profile derives its model requirements from the Filters it
contains.

``` text
Profile
│
├── System Prompt Filter
│      requires system-message semantics as defined by the Filter
│
└── Cognition Filter
       requires known usable context characteristics
```

Moderari is authoritative for Filter/Profile applicability.

Praebere supplies model truth and may mediate the applicability query
during session configuration.

Rogare displays the result but does not reproduce Filter requirement
logic.


## 12.1 Client Representation Is a First-Ask Execution Requirement

Filter Profile model applicability can be evaluated during session configuration because Praebere has already supplied the reserved provider/model and Effective Model Definition. The client interaction representation may not be known until the first ordinary Ask reaches Moderari.

A profile may therefore be selected before client-representation recognition, but that selection is conditional on first-Ask representation compatibility. Selecting a Filter Profile creates a positive representation requirement: Moderari must be able to recognise the client representation and construct the Filter loopback/provider execution path required by that session.

``` text
First Ask
   │
   ▼
recognise client representation
   │
   ├── Unknown + No Filters ─────────► pass through
   ├── Unknown + Filters selected ───► reject
   └── Known
          │
          ▼
      compatible client↔provider path?
          ├── yes ───────────────────► execute
          └── no ────────────────────► reject
```

> **Unknown representation does not inherently prevent Vanilla execution; it prevents intentional Filter transformation.**

> **A selected Filter Profile may execute only when Moderari can positively establish the client representation and the compatible representation path required to materialise Final Filter Output.**

# 13. Plugin Trust and Dependencies

A third-party Filter loaded in-process executes code inside Moderari.

Installation and enablement are therefore trust decisions.

Extension discovery and extension loading are separate stages.

During discovery, Moderari inspects the extension package and its
manifest without requiring the extension's executable Python code to be
imported.

Once an extension has been explicitly trusted/enabled, Moderari loads
and registers the extension during startup.

Conceptually:

```text
Moderari startup
       │
       ▼
Discover Filter package
       │
       ▼
Inspect and validate manifest
       │
       ▼
Filter trusted/enabled?
       │
   ┌───┴───┐
   │       │
  NO      YES
   │       │
   │       ▼
   │   Load extension
   │       │
   │   ┌───┴──────────────┐
   │   │                  │
   │  success            failure
   │   │                  │
   │   ▼                  ▼
   │ Register Filter   Moderari startup
   │                   failure
   ▼
Do not load

# 14. Development Workflow

A researcher should be able to work in an independent source repository:

``` text
C:\Development\My-Lumen-Filter
        │
        │ Docker bind mount
        ▼
/lumen/extensions/filters/my-filter
        │
        ▼
Moderari discovery
```

The Research Foundation image remains unchanged.

Restart-based discovery is sufficient for the initial implementation.
Dynamic Python hot-reload is outside current scope.

# 15. Filter Direction and Execution Records

Moderari Filters are strictly **to-model** transformations.

``` text
Client interaction/context
          │
          ▼
      Filter(s)
          │
          ▼
        Model
```

A Filter may modify the context presented to the model, including System
Prompt content, context selection, cognition snapshots or future
researcher-defined transformations.

A Moderari Filter does not transform the model response.

This is a deliberate architectural restriction. It preserves an
unambiguous client-visible response for the Vestigare trace and prevents the
Filter framework from creating a second client-facing answer.

If response transformation is required, it belongs outside the Moderari
Filter framework and may be performed by the client, agent or another
external component.

## 15.1 Filter Execution Identity and Persisted Results

Moderari owns Filter execution identity. The Filter Framework creates a unique `filter_execution_id` each time it invokes a Filter. A Filter implementation does not create or infer its own execution identity.

Moderari knows whether material came from the client, a Filter or a model from the execution path that produced it. Origin must not be inferred from message content, role or interaction representation, and Lumen provenance metadata must not be inserted into the external client representation.

For every Filter execution, Moderari persists the **Final Filter Output** in the recognised client interaction representation and associates it with the `filter_execution_id`, Filter identity/version and effective configuration.

A Filter may itself invoke a model. The current built-in example is Cognition. Any private model Ask and model Response used by that Filter are also persisted as Filter execution artefacts associated with the same `filter_execution_id`. The returned model response is persisted after normal provider-to-client representation processing, so it is in the client interaction representation.

``` text
filter_execution_id = F123
        │
        ├── Filter identity/version/configuration
        ├── private model Ask       # where applicable
        ├── private model Response  # where applicable
        └── Final Filter Output     # always client representation
```

Only the Final Filter Output is the persisted output of the Filter and the material incorporated into session context. Private model interactions are supporting execution artefacts; they do not become Vestigare raw conversation turns.

> **Interaction representation describes what the material is. Moderari execution provenance describes where it came from.**

## 15.2 Vestigare Does Not Store Filter Transformations

Vestigare records:

``` text
exact client interaction/context presented to Lumen
exact response returned to the client in the client interaction representation
```

It does not store the Filter-transformed model context as part of the
raw conversation trace.

A Filter transformation may occur anywhere in construction of the
model-facing interaction. Multiple Filters may run, and a Filter may
participate more than once depending on its execution contract.

Trying to embed those intermediate states in Vestigare would make the
raw trace dependent on the installed Filter framework.

Therefore:

> **Vestigare records what was said. Moderari records what it did.**

## 15.3 Session Execution Metadata and Filter Transformations

When the Moderari Session State is materialised, the relevant execution
environment metadata is sent to Vestigare using the same
execution-metadata facility that already records facts such as the model
used.

This may include:

``` text
resolved Filter IDs
Filter implementation versions
effective Filter configuration
execution order/stage
Adapter state/version where applicable
other relevant Moderari session setup
```

This metadata describes the environment under which the trace occurred.
It does not become part of the raw conversation trace.

The actual intermediate text/context transformations performed by
Filters are a different concern. They are not inserted into the
Vestigare raw conversation. Where such transformation records are
retained for research, they remain separate from the raw trace and can
be correlated later by Aestimare or external research tooling.

> **Vestigare records what was said; its execution metadata records the
> environment. Filter transformation records describe what Moderari
> did.**

------------------------------------------------------------------------

# 16. Filter Version and Code Identity

Filter implementations may change while a Filter Profile remains
conceptually the same.

Vestigare execution metadata should therefore be able to identify the
Filter implementation configured for the materialised Moderari session,
including:

``` text
Filter ID
implementation version
effective configuration
execution stage/order
package/code fingerprint where available
```

Filter Profiles themselves do not require historical versions.

# 17. Repetere Replay Boundary

Repetere replays the recorded client-boundary interaction within a
Moderari execution environment.

There are two supported ways to establish that environment:

``` text
Recorded Client-Boundary Interaction
                │
                ▼
           Repetere Replay
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
 ORIGINAL SETUP       NEW SETUP
        │                │
        ▼                ▼
Recorded Session      New Session
Execution State       Execution Plan
        │                │
        ▼                ▼
Reconstruct           Resolve normally
original              through Lumen
environment           session configuration
        │                │
        └───────┬────────┘
                ▼
          Replay interaction
```

## 17.1 Original Setup

For an **Original Setup replay**, Repetere uses the recorded Moderari
Session Execution State preserved with the original execution metadata.

That state is the execution truth for the original session and contains
the information required to reconstruct the Moderari environment used
for that execution, including the resolved Filter implementations,
versions and complete effective configuration, together with the
effective Adapter implementation/state.

Repetere requests that Moderari establish that recorded execution
environment before submitting the first recorded Ask.

Moderari validates whether the recorded execution path can still be
constructed from the currently available execution components. This includes
the recorded Filter implementations/versions and Adapter
implementation/version.

If the required execution path is still available, Moderari reconstructs it
and replay proceeds. If a required recorded Filter or Adapter
implementation/version is no longer available, Moderari rejects the Original
Setup. It must not silently substitute a newer or different implementation.

The user may then create a **New Setup**, which is resolved normally against
the Filters, Adapters and other execution components currently available.

The reconstruction does not depend upon current Filter Profiles, saved
System Prompts or other mutable configuration-time resources. Those resources
are configuration conveniences, not historical execution dependencies.

## 17.2 New Setup

For a **New Setup replay**, Repetere does not modify or selectively
override the recorded Session Execution State.

Instead, it creates a new session execution plan through the normal
Lumen session-configuration process.

The user may select a new model and a Moderari Filter Profile, including
explicit `None`.

Praebere and Moderari then resolve the resulting execution environment
normally, including:

- the selected provider/model execution target;
- Filter applicability;
- the resolved effective Filters and their configuration; and
- the effective Adapter required for execution.

A new Moderari Session Execution State is materialised from that
execution plan before the recorded interaction is replayed.

Changing the model may itself change Filter applicability and Adapter
resolution. Those dependent execution decisions are therefore resolved
afresh rather than inherited from the original execution.

## 17.3 No Mutation of Recorded Execution State

Repetere does not construct comparative replays by changing individual
properties of the recorded Session Execution State.

``` text
NOT:

Recorded Execution State
       │
       ├── change Model
       ├── remove Filter
       ├── replace System Prompt
       └── change Adapter
              │
              ▼
         modified state


INSTEAD:

Original Setup                  New Setup
      │                             │
      ▼                             ▼
reconstruct recorded          create new session
execution state               execution plan
      │                             │
      └─────────────┬───────────────┘
                    ▼
             replay same recorded
          client-boundary interaction
```

Selecting no Filters is therefore simply a New Setup with the Moderari
Filter Profile explicitly set to `None`.

The Adapter is not independently selected by Repetere. For a New Setup,
it is resolved normally by Moderari from the newly established execution
environment.

> **Replay source material is the recorded client-boundary interaction.
> A replay either reconstructs the original recorded Session Execution
> State or creates a new Session Execution State from a new session
> execution plan. Repetere does not mutate the recorded Session
> Execution State.**

# 18. Current Built-In Filters

The initial implementation contains exactly:

``` text
System Prompt Filter
Cognition Filter
```

Additional Filters are expected to come from future research and
extension work rather than from expanding the current redesign.

# 19. Filter Requirements

## MFR-01 --- Discoverable Extensions

A compliant Filter can be installed outside Moderari Core and discovered
through the configured extension location.

## MFR-02 --- Self Description

A Filter declares identity, implementation version, requirements,
configuration schema and execution stage through a machine-readable
contract.

## MFR-03 --- Textual/Contextual Interaction

Filters operate on Moderari-managed textual/contextual interaction
material. They do not depend on the CIM and must not require
protocol-specific parsing to express their intentional transformation.

## MFR-04 --- Session Scope

Effective Filter configuration is session-scoped.

## MFR-05 --- Explicit Selection

Filters are applied only through an explicit session Filter Profile
decision. Zero Filters is valid.

## MFR-06 --- Moderari Authority

Moderari performs authoritative Filter/Profile applicability and
configuration validation.

## MFR-07 --- Praebere Model Truth

Filter requirements are evaluated against the Effective Model Definition
supplied by Praebere. Filters do not maintain competing model knowledge.

## MFR-08 --- Unknown Capability

An unknown model characteristic does not satisfy a positive Filter
requirement.

## MFR-09 --- Servire Administration

Detailed Filter discovery, description, configuration and Filter Profile
editing belong to the Moderari UI in Servire. External clients need only
consume selectable session configuration.

## MFR-10 --- Open Extension Boundary

A new compliant Filter must not require Filter-specific code changes to
Rogare, Vestigare or Repetere.

## MFR-11 --- Execution Metadata

The resolved Filter implementation/version/configuration for a
materialised Moderari session must be available as Vestigare execution
metadata without placing Filter-transformed context into the Vestigare
raw conversation.

## MFR-12 --- No Historical Profile Dependency

Replay/provenance must depend on the resolved effective Filter
environment, not historical Filter Profile versions.

## MFR-13 --- To-Model Direction Only

Moderari Filters may transform only the interaction/context sent toward
the model. They must not transform the client-visible response.

## MFR-14 --- Current Context Ownership

For the current redesign Moderari retains the session
conversation/model-facing context required by the existing interaction
and Filter machinery. Moving full context ownership to clients such as
Rogare is a deferred requirement.

## MFR-15 --- No Default System Prompt

Moderari supplies no implicit default System Prompt. A Lumen/user System
Prompt is selected through a Filter Profile containing the System Prompt
Filter.

## MFR-16 --- Filter Profile Validity

A Filter Profile containing one or more unavailable Filters remains a
persistent, editable Moderari configuration resource but is invalid for
execution. It must not be included in the selectable Filter Profiles exposed
to clients. Moderari administration must identify the unavailable Filter(s).

## MFR-17 --- Original Setup Availability

For Original Setup replay, Repetere requests the recorded Session Execution
State and Moderari determines whether the recorded execution path can still be
constructed. If a required recorded Filter or Adapter implementation/version is
unavailable, Moderari must reject the Original Setup rather than substitute a
different implementation. The user may create a New Setup through normal
session configuration.

## MFR-18 --- First-Ask Representation Compatibility

Filter Profile validity and model applicability are evaluated during session
configuration. A selected Filter Profile is executable only after Moderari
positively recognises the client interaction representation on the first Ask
and establishes a compatible representation path to the known provider
representation.

An `Unknown` client representation with selected Filters must cause the first
Ask to be rejected. An `Unknown` client representation with No Filter Profile
may use Vanilla pass-through. If client and provider representations are both
known but no compatible Adapter path exists, the Ask must be rejected.

## MFR-19 --- Filter Execution Identity and Final Output

Moderari's Filter Framework must create and own a unique `filter_execution_id` for each Filter invocation. The Filter Framework must treat the result after any required loopback translation as the **Final Filter Output**, persist that output in the recognised client interaction representation, and associate it with the Filter execution identity.

## MFR-20 --- Private Model Interaction Evidence

Where a Filter invokes a model as part of its execution, Moderari must persist the private model Ask and returned model Response as Filter execution artefacts associated with the same `filter_execution_id`. These artefacts must remain separate from the Vestigare raw conversation trace.

## MFR-21 --- Provenance Is Execution-Path Knowledge

Moderari must determine whether interaction material originated from the client, a Filter or a model from the execution path that produced it. Origin must not be inferred from message content, role or representation, and provenance metadata must not be inserted into the external interaction representation.

# 20. Acceptance Criterion

> **A new Moderari Filter should be independently installable and
> discoverable; declare its own requirements, output representation and
> configuration contract; produce interaction material without constructing
> or consuming the CIM; participate automatically in Filter Profile
> applicability against Praebere's Effective Model Definition; be administered
> through the Moderari UI in Servire; produce a Final Filter Output in the
> recognised client interaction representation; have its execution identity
> and results persisted generically by Moderari; and require no Filter-specific
> changes to Rogare, Vestigare or Repetere.**


# 20. 02026-10-02 Clarifications — Client Representation, Evidence and Replay

## 20.1 Filter Working Domain

A Filter's effective working domain is the session's recognised **client interaction representation**. Filters do not operate in the provider representation and do not construct or consume the CIM.

A Filter creates material in one supported external representation that it knows how to construct. The Filter Framework then obtains the **Final Filter Output** in the recognised client representation, using Adapter Framework loopback when the Filter output representation differs. The CIM, where needed for that translation, remains internal to the Adapter Framework.

The Final Filter Output is what Moderari persists and incorporates into session context. The Filter Framework creates and owns the `filter_execution_id` that correlates the Filter invocation, its Final Filter Output and any private model Ask/Response generated as part of that execution.

## 20.2 Cognition

The built-in Filter is named **Cognition**. Cognition does not decide which historical messages are important enough to compact. Moderari applies the configured context-consumption trigger; Cognition then performs a private Ask requesting the model's present understanding at that point in time. The returned cognition is used with retained actual context to form replacement model-facing context.

The exact effect of this replacement on client-maintained or client-cached context remains an implementation-discovery item.

A Cognition private Ask is an internal Moderari operation. It may use the client-representation loopback facility and the normal Adapter/provider execution path, but it does **not** create a Vestigare conversation-trace entry.

## 20.3 Vestigare Trace Authority

Filters have no API for adding, removing, replacing or suppressing Vestigare raw interaction records. Adapter extensions have the same restriction.

Vestigare records the interaction at the **Client ↔ Lumen boundary**:

``` text
Ask      = interaction/context actually received from the client,
           in the client interaction representation

Response = interaction actually returned to the client,
           in the client interaction representation
```

If a provider uses a different representation, the provider-side response is translated before the response is recorded. Vestigare does not retain the transient provider representation as the conversation response. Adapters change representation, not semantic interaction content.

Filter activity affects the raw trace only when its consequences naturally propagate through the ordinary Model → Lumen Stack → Client interaction and therefore become part of what is actually returned to the client. Private Filter operations such as Cognition do not themselves create trace entries.

## 20.4 System Prompt and Removal of the Legacy Replay Backchannel

The legacy System Prompt replacement backchannel to Vestigare is no longer required by the redesigned session execution model. It is a migration item to remove.

The original client Ask remains in the Vestigare trace, including any System Prompt supplied by the client. On first Ask, Moderari materialises a self-contained Session Execution State containing the actual System Prompt Filter implementation/version and the complete effective System Prompt content/configuration used for that session. That materialised state is retained as Vestigare execution metadata.

The Session Execution State must not depend on a Filter Profile name, saved System Prompt name, or any other mutable Moderari configuration resource. Those resources may later be edited, renamed or deleted without changing replay truth.

For an **Original Setup replay**, Repetere reconstructs Moderari from the recorded Session Execution State **before** replaying the first recorded Ask. The System Prompt Filter then performs the same replacement from its recorded effective configuration. For a **New Setup replay**, Repetere instead establishes a new session execution plan through the normal Lumen configuration path and a new Session Execution State is materialised before replay begins. Repetere no longer requires the legacy System Prompt pass-through replay mode or special replacement headers.

> **Filter Profile is configuration-time convenience. Materialised Session Execution State is execution-time truth.**
