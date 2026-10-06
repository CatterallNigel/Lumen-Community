# Lumen Praebere, Rogare, Pontis, Vestigare and Repetere Companion Changes

## Supporting the Moderari Redesign

## 1. Purpose

The Moderari redesign establishes a stronger separation of
responsibility between Praebere, Moderari and Rogare.

Moderari does not own persistent model knowledge. During session
configuration, Praebere reserves the selected provider/model for the Pontis
session and supplies Moderari with the authoritative Effective Model Definition,
provider interaction representation and provider/model execution information
required for that reservation. Moderari retains those facts as session-scoped
execution configuration, evaluates Filter Profile applicability, and adds the
user's resolved Filter Profile decision before the first Ask.

The first Ask is therefore not a second model-resolution step. It supplies the
request-boundary evidence from which Moderari attempts to recognise the client
interaction representation and resolve the effective Adapter path. The Filter
Profile is configuration-time intent; the resolved Filter set is execution-time
truth.

This creates corresponding architectural requirements for Praebere and
Rogare.

This document defines those companion changes.

------------------------------------------------------------------------

# 2. Responsibility Boundary

The services have distinct responsibilities. Pontis owns the session and
request/response routing; Praebere owns provider/model knowledge and the
session-to-model reservation; Moderari owns interaction transformation;
Rogare presents and configures the available choices.

The principal questions are:

``` text
PRAEBERE
"What am I running, and what do I know about it?"

MODERARI
"Given that model, which transformations are valid?"

ROGARE
"What can the user select and configure?"
```

Pontis owns client request/response transport and session routing, but
does **not** need to classify the client's AI interaction representation.

Moderari's Adapter Framework determines the client interaction representation
from the request boundary using recognition/validation knowledge supplied by
installed representation Adapters. Praebere remains authoritative for the
provider interaction representation and provider/model knowledge.

This creates the Adapter authority boundary:

``` text
Pontis
    request + session transport
           │
           ▼
Moderari Adapter Framework
    determine client interaction representation
    resolve installed Adapter
           ▲
           │
Praebere
    provider interaction representation
    + provider/model characteristics
```

Neither Pontis nor Praebere knows which Adapter implementations are installed.
If Moderari cannot positively distinguish the client representation, it
retains Unknown rather than guessing.

The intended session-configuration flow is:

``` text
                    ROGARE
                       │
                       ▼
                    Pontis
                       │
                 Create Session
                       │
                       │ session_id
                       ▼
                    Rogare
                       │
                User selects model
                       │
                       │ session_id + model
                       ▼
                    Praebere
                       │
             Reserve model for session
             Resolve model definition
                       │
                       ├──────────────► Moderari
                       │                 │
                       │          Evaluate Filter Profile
                       │              requirements
                       │                 │
                       ◄─────────────────┘
                       │
          model + profile applicability
                       │
                       ▼
                    Rogare
                       │
             User selects profile
                       │
                       ▼
                   Moderari
                       │
          Configure existing session
                       │
                       ▼
                    Rogare
                       │
                   First Ask
```

Neither Rogare nor Moderari should maintain a competing model catalogue.

Praebere is the sole authority for the effective provider/model
definition used by a Lumen session.

During configuration of an existing Pontis session, Praebere may ask
Moderari to evaluate Filter Profile applicability for a selected model
and relay Moderari's result to Rogare. This does not make Praebere an
authority for Moderari Filters or Filter Profiles.

------------------------------------------------------------------------

# 3. Praebere Model and Provider Knowledge

Praebere must describe the selected execution target sufficiently for
other Lumen services to make safe decisions.

The execution definition now needs to distinguish the **provider
interaction representation** from **model capabilities**.

Conceptually:

``` yaml
provider:
  id: ollama
  interaction_representation: openai-compatible

model:
  id: qwen2.5-coder:14b
  family: qwen2.5-coder
  context_window: 32768

  capabilities:
    system_messages: true
    tools: true
```

The provider representation is important to Moderari Adapter resolution.
It should not be described as "the protocol the model uses"; the
provider endpoint normally owns the wire/API representation.

The capability vocabulary must be extensible. Lumen should define
characteristics only when an actual consumer requires them rather than
attempt to enumerate every possible future model capability.

Potential capability families include:

``` text
interaction
    system messages
    multi-turn semantics

tools
    declarations
    calls
    results
    parallel calls

output
    structured output
    JSON output

context
    maximum/effective context characteristics

provider/execution
    streaming
    interaction representation
```

The exact schema remains an implementation task.

Runtime policy, Filter configuration, Adapter identities and provider
operational settings do not belong in the model definition.

Praebere has no knowledge of installed Moderari Adapters or Filters.

------------------------------------------------------------------------

# 3.1 Praebere Owns the Session-to-Model Reservation

The move away from one globally selected model requires Praebere to own
an explicit provider/model reservation for each Pontis session.

``` text
Pontis
    session-A
    session-B
    session-C

Praebere
    session-A -> Ollama -> qwen2.5-coder:14b
    session-B -> Ollama -> gemma3:4b
    session-C -> Ollama -> qwen2.5-coder:14b
```

Pontis remains authoritative for the existence and lifecycle of the
session itself. Praebere is authoritative for which provider/model is
reserved for that session and for provider/model residency and
ownership.

The current global selected-model assumption must therefore be replaced
by per-session reservations. Multiple sessions may reserve the same
model or different models concurrently.

When a session ends, Pontis informs Praebere. Praebere removes that
session's reservation and applies its model lifecycle rules. If no other
session is using the model and Praebere owns the model residency,
Praebere may unload it. If another session still uses the model, or
Praebere does not own its residency, it must not unload it merely
because one session ended.

``` text
Pontis
  │
  │ session ended
  ▼
Praebere
  │
  ├── identify reserved provider/model
  ├── release session reservation
  ├── other sessions using model?
  │      ├── yes -> retain
  │      └── no
  │           └── Praebere owns residency?
  │                 ├── yes -> unload according to policy
  │                 └── no  -> leave resident
  └── complete release
```

This makes provider/model lifecycle a Praebere concern rather than a
Pontis model-state concern.

------------------------------------------------------------------------

# 3.2 First-Ask Activation and Session Materialisation

The first ordinary Ask retains the existing provider/model readiness gate, but it no longer causes Moderari to ask Praebere for model knowledge already supplied during session configuration.

When Rogare selects a model for an existing Pontis `session_id`, Praebere creates and retains the session-to-provider/model reservation, resolves the Effective Model Definition and provider interaction representation, and supplies the required authoritative execution/model information to Moderari. The reservation remains authoritative until the session ends. Moderari retains those facts as session-scoped execution configuration; it does not maintain a second persistent model library.

Praebere and Moderari then support Filter configuration:

``` text
Praebere
  │ authoritative reserved-session facts
  ▼
Moderari
  │ evaluate valid/model-applicable Filter Profiles
  ▼
Rogare
  │ select Profile or None
  ▼
Moderari
  │ resolve Filters + configuration
  └── session configuration fixed
```

On the first Ask, Pontis still activates the execution session through Praebere. Praebere uses the existing reservation, ensures provider/model readiness, loads the model where required, and confirms readiness. Moderari then attempts to recognise the client interaction representation from the request boundary.

``` text
FIRST ASK

recognise client representation
        │
        ├── Unknown
        │      ├── No Filters ─────────► pass through
        │      └── Filters selected ───► reject first Ask
        │
        └── Known
               │
               ▼
        resolve client ↔ provider path
               │
               ├── compatible ─────────► execute
               └── incompatible ───────► reject first Ask
```

After successful resolution, the client representation and effective Adapter path become part of the fixed Moderari Session Execution State for the remainder of the session.

> **Praebere knows and reserves the execution target. Moderari configures the session from those authoritative facts. The first Ask establishes only request-boundary execution facts that could not have been known earlier.**

# 4. Model Characteristics Require an Unknown State

Praebere must not convert absence of information into a negative
assertion.

For capability-style characteristics, the effective state is
conceptually:

``` text
true
false
unknown
```

For example:

``` yaml
capabilities:
  system_messages: true
  tools: unknown
  streaming: true
```

The meanings are:

``` text
true
    Praebere has sufficient information that the capability is supported.

false
    Praebere has sufficient information that the capability is not supported.

unknown
    Praebere does not currently have sufficient information to establish
    whether the capability is supported.
```

The distinction is essential because Moderari uses these characteristics
when determining whether filters and profiles are applicable.

Unknown must therefore remain explicit throughout the execution-target
contract.

------------------------------------------------------------------------

# 5. Praebere Model Catalogue

Provider discovery cannot be assumed to provide a complete model
description.

Praebere therefore needs a persistent **Model Catalogue**.

The catalogue belongs to Praebere because provider/model knowledge is
Praebere's responsibility.

Conceptually:

``` text
Praebere
│
├── Provider Registry
│
├── Provider Discovery
│
├── Model Catalogue
│   ├── Lumen-supplied model definitions
│   └── User-supplied model definitions / overrides
│
└── Effective Model Resolution
```

The catalogue must not be a hard-coded Python mapping.

Researchers must be able to add definitions for models that Lumen does
not yet know about and supplement incomplete information for discovered
models.

For example:

``` yaml
model:
  id: experimental-model-v7
  family: experimental

  context_window: 65536

  capabilities:
    system_messages: true
    tools: false
    streaming: true
```

------------------------------------------------------------------------

# 6. Built-In Definitions and User Definitions Must Be Separate

Lumen-supplied catalogue information and user-maintained information
should be stored separately.

This prevents a Lumen upgrade from overwriting researcher configuration.

Conceptually:

``` text
Lumen-supplied Catalogue
          +
Provider-discovered Facts
          +
User Definitions / Overrides
          │
          ▼
Effective Model Resolution
          │
          ▼
Effective Model Definition
```

The exact precedence rules require implementation design.

In particular, Praebere should be cautious about allowing a user
override to silently contradict a characteristic authoritatively
reported by the active provider.

Regardless of precedence, the source of each effective characteristic
must remain identifiable.

------------------------------------------------------------------------

# 7. Characteristic Provenance

Praebere should preserve where each characteristic came from.

Initial source categories may include:

``` text
DISCOVERED
    Reported by the provider/runtime.

CATALOGUE
    Supplied by the Lumen model catalogue.

USER
    Supplied or overridden by the researcher.

INFERRED
    Derived by Praebere rather than directly established.
```

`INFERRED` should be used cautiously and must never be presented as an
established fact.

Conceptually:

``` text
context_window = 32768
source = discovered

tools = true
source = catalogue

system_messages = true
source = user
```

This allows Lumen to distinguish what is known from what has merely been
declared or inferred.

------------------------------------------------------------------------

# 8. Effective Model Definition

Praebere combines the available sources into one authoritative effective
model definition.

``` text
Provider Discovery
        │
        ├─────────────┐
        │             │
        ▼             ▼
Lumen Catalogue   User Definition
        │             │
        └──────┬──────┘
               ▼
       Resolution / Validation
               │
               ▼
     Effective Model Definition
```

Moderari and other Lumen services consume this effective definition
rather than reproducing the resolution logic themselves.

The execution-target contract should include sufficient identity and
provenance to allow the definition used for a session to be traced.

------------------------------------------------------------------------

# 9. Unknown and Incomplete Models

Praebere must permit a provider to expose a model that is not present in
the Lumen catalogue.

Such a model should not be rejected merely because its definition is
incomplete.

Instead it can appear as:

``` text
experimental-model-v7

Availability: available
Definition: incomplete

Context window: unknown
System messages: unknown
Tools: unknown
Streaming: discovered true
```

The researcher can then add or edit the missing model definition.

This supports local and experimental models without requiring a Lumen
software release for every new model.

------------------------------------------------------------------------

# 9.2 One User Model Profile Per Model

Every discovered model may have at most one **Praebere User Model
Profile**.

The User Model Profile is not a competing complete definition of the
model.

It is a persistent gap-filling resource containing information supplied
by the researcher where Praebere could not establish that information
automatically.

Conceptually:

``` text
Praebere Model
│
├── Provider-discovered Information
├── Lumen Catalogue Information
└── User Model Profile
       └── zero or one per model
            │
            ▼
    Effective Model Definition
```

The User Model Profile should not require the researcher to re-enter
information that Praebere already knows.

For example:

``` text
Model: qwen36-35b-a3b

Characteristic          Value       Source
────────────────────────────────────────────
Family                  qwen3.6     Catalogue
Maximum Context         Unknown     —
System Messages         Yes         Catalogue
Tools                   Unknown     —
Streaming               Yes         Provider
```

If only maximum context remains unknown, the User Model Profile may
contain only:

``` yaml
model:
  id: qwen36-35b-a3b

user_profile:
  context_maximum: 32768
```

The resulting effective definition combines that declaration with the
other known sources.

The governing principle is:

> **Praebere should never require the user to provide model information
> that it can reliably discover or verify itself.**

The User Model Profile is therefore a gap-filling mechanism, not the
primary model configuration mechanism.

------------------------------------------------------------------------

# 9.3 Incomplete Model Definitions Remain Usable

A model with unknown characteristics must not automatically become
unusable.

Praebere should distinguish model availability from model-definition
completeness.

For example:

``` text
qwen36-35b-a3b

Availability: Available
Definition: Incomplete

Maximum context: Unknown
System messages: Yes
Tools: Unknown
Streaming: Yes
```

The model can still be selected.

Moderari determines which transformations are safe given the
characteristics that are known.

For example:

``` text
Vanilla                  ✓ available
Custom System Prompt     ✓ available
Compaction               unavailable — context unknown
Tool Research            unavailable — tools unknown
```

Therefore:

> **Praebere reports what is known and unknown; Moderari determines
> which profiles are applicable to that knowledge.**

An incomplete definition is informational and capability-limiting, not
an automatic execution prohibition.

------------------------------------------------------------------------

# 9.4 Model Discovery Becomes an Active Resolution Process

Praebere model discovery should become more than obtaining a list of
model names.

For each discovered model, Praebere should attempt to construct the
strongest evidence-backed effective definition available.

Conceptually:

``` text
Provider
   │
   ▼
Discover Models
   │
   ▼
For each model
   │
   ├── Gather provider metadata
   ├── Match Lumen catalogue
   ├── Load User Model Profile if present
   │
   ▼
Resolve Effective Model Definition
   │
   ├── COMPLETE
   │
   └── INCOMPLETE
          │
          ▼
      Highlight in UI
```

provider/model combination rather than an uncontrolled battery of
inference requests.

The resulting model record should expose both:

``` text
Availability
Definition completeness
```

These are independent states.

------------------------------------------------------------------------

# 9.5 Terminology: Praebere Model Profile vs Moderari Filter Profile

Lumen now has two deliberately different uses of the word `Profile`.

## Praebere User Model Profile

> **User-maintained knowledge about one particular model, used to fill
> gaps in Praebere's effective model definition.**

Properties:

``` text
zero or one per model
model-specific
knowledge/configuration resource
fills unknown characteristics
owned by Praebere
```

## Moderari Filter Profile

> **A named, reusable interaction-transformation configuration that
> resolves to a session-scoped filter set.**

Properties:

``` text
many may exist
many may apply to one model
contains/resolves filters
defines interaction transformation
owned by Moderari
```

The relationship is:

``` text
                    MODEL
                      │
                      ▼
          Praebere User Model Profile
             "What do we know?"
                      │
                      ▼
          Effective Model Definition
                      │
                      ▼
                  Moderari
                      │
             "What can be applied?"
                      │
                      ▼
          Applicable Moderari Filter Profiles
                      │
                      ▼
                   Rogare
```

The two profile types must remain separate in APIs, UI terminology and
persistence.

------------------------------------------------------------------------

# 10. Praebere User Interface Changes

Praebere's UI should evolve from a simple model list into a model
knowledge view.

A model may conceptually display:

``` text
Qwen 2.5 Coder 14B
Provider: Ollama
Status: Available

Characteristic          Value       Source
────────────────────────────────────────────
Context Window           32768       Provider
System Messages          Yes         Catalogue
Tools                    Yes         Catalogue
Streaming                Yes         Provider
```

For an incomplete model:

``` text
Experimental Model V7
Provider: Colibri
Status: Available
Definition: Incomplete

Characteristic          Value       Source
────────────────────────────────────────────
Context Window           Unknown     —
System Messages          Unknown     —
Tools                    Unknown     —
Streaming                Yes         Provider

[Edit Model Definition]
```

Praebere should support, at minimum:

``` text
View model definition
Add user model definition
Edit user-maintained metadata
Remove user-maintained metadata
Show characteristic source/provenance
Identify incomplete definitions
```

Editing the Lumen-supplied catalogue directly should not be required.

------------------------------------------------------------------------

# 11. Praebere API / Control Requirements

Praebere must expose enough information for the rest of Lumen to obtain:

``` text
available providers
available models
effective model definition
characteristic values
unknown characteristics
characteristic provenance
definition completeness
```

The exact API and `\obt` command syntax remain to be designed.

The important architectural rule is:

> **Consumers request the effective model definition from Praebere; they
> do not reconstruct it independently.**

------------------------------------------------------------------------

# 12. Moderari Compatibility Contract

Moderari consumes two different kinds of authoritative compatibility
information.

For Adapters:

``` text
Pontis
    request + session transport
         │
         ▼
Moderari Adapter Framework
    determine client representation
         +
Praebere
    provider representation
    model characteristics
         │
         ▼
Moderari Adapter resolution
```

Adapters translate between external representations and Moderari's
internal Canonical Interaction Model. The Canonical Interaction Model is
not a new Lumen network protocol.

For Filters:

``` text
Praebere
    Effective Model Definition
          │
          ▼
Moderari
    Filter requirements
          │
          ▼
Filter/Profile applicability
```

Praebere does not determine Filter applicability and does not know
installed Adapters/Filters.

Moderari does not reconstruct model truth. Its Adapter Framework may inspect
the request boundary against installed, defined representation contracts to
determine the client representation, but it must not guess from ambiguous or
undocumented payload features.

Detailed contracts are defined in the separate **Lumen Moderari Adapter
Architecture** and **Lumen Moderari Filter Architecture** documents.

------------------------------------------------------------------------

# 13. Rogare Model-First Session Configuration

Rogare should use model selection as the first constraint on Moderari
Profile selection.

The normal session-creation flow becomes:

``` text
1. Rogare creates the session through Pontis
2. Pontis returns the session ID
3. Rogare selects provider/model for that session through Praebere
4. Praebere reserves the provider/model for the session
5. Praebere resolves the Effective Model Definition
6. Praebere asks Moderari for Filter Profile applicability
7. Moderari returns applicability to Praebere
8. Praebere returns selected-model information and applicability to Rogare
9. Rogare selects an applicable Moderari Filter Profile for the session
10. Moderari configures that existing session
11. First Ask
```

The user should not need to create an active execution session merely to
discover which profiles are usable.

------------------------------------------------------------------------

# 14. Rogare Receives Filter Profile Applicability Through Model Selection

Rogare does not calculate Filter Profile applicability and does not ask
Moderari directly for applicable Filter Profiles.

Filter Profile applicability is returned to Rogare as part of the
Praebere-mediated model-selection flow.

When Rogare selects a provider/model for an existing Pontis session,
Praebere reserves that provider/model and resolves its authoritative
Effective Model Definition.

Praebere supplies that model information to Moderari so that Moderari can
evaluate its currently available Filter Profiles against the selected model.

Conceptually:

``` text
Rogare
   │
   │ select model + session_id
   ▼
Praebere
   │
   ├── reserve provider/model
   ├── resolve Effective Model Definition
   │
   │        model facts
   │            ▼
   │        Moderari
   │            │
   │            ├── evaluate Filter Profiles
   │            └── return applicable Profiles
   │            │
   ◄────────────┘
   │
   │ model selection result
   │ + applicable Filter Profiles
   ▼
Rogare
```

Praebere does not calculate Filter Profile applicability and acquires no
knowledge of Filter or Filter Profile semantics. It supplies the authoritative
model facts to Moderari and relays Moderari's applicability result to Rogare
as part of the model-selection response.

Rogare then presents the applicable Filter Profiles together with the explicit
`No Filter Profile` choice.

When the user makes that choice, Rogare sends the Filter Profile decision for
the existing `session_id` to Moderari through Nuntius using `\obt`:

``` text
Rogare
   │
   │ user selects Filter Profile / None
   │
   │ \obt moderari profile select
   │ session_id=<session_id>
   │ profile_id=<profile | None>
   ▼
Nuntius
   │
   ▼
Moderari
   │
   └── configure session Filter Profile
            │
            ▼
      SESSION CONFIGURED
            │
            ▼
   Rogare ready for First Ask
```

Rogare therefore requires no model-characteristic, Filter-requirement or
Filter-applicability logic of its own.

The architectural flow is:

> **Rogare selects the model through Praebere. Praebere returns the applicable
> Filter Profile choices obtained from Moderari. Rogare sends the user's Filter
> Profile decision to Moderari through `\obt`. Once that decision is complete,
> Rogare is ready for the first Ask.**


------------------------------------------------------------------------

# 15. Context Ownership and System Prompt Selection

For the current redesign, Moderari continues to retain the session conversation/model-facing context required by the existing interaction and Filter machinery.

Moving complete conversational-context ownership to clients such as Rogare is a later requirement. That future design must define how client-owned raw context is reconciled with Moderari-managed transformed/model-facing context.

Moderari has no default System Prompt. Any Lumen/user System Prompt must be selected through a Moderari Filter Profile containing the System Prompt Filter.

A Custom System Prompt therefore becomes a Filter Profile in its own right. The user may add another Filter to that profile if required.

Rogare's interaction controls remain disabled until the model and explicit Filter Profile decision have been made. `No Filter Profile` remains a valid decision and means no intentional Moderari Filters.

# 15A. Provider Tools Is Separate from Filter Applicability

Rogare's existing **Provider Tools** checkbox is a client/session
control used to indicate whether provider-tool description material
should be removed or omitted from the System Prompt. The implementation
work must confirm whether that behaviour currently resides in Moderari
or Pontis and preserve the existing owner unless deliberately changed.

This is separate from Moderari Filter applicability. If a model does not
satisfy the requirements of a Tool Use Filter, Moderari simply does not
make that Filter applicable. Other external clients remain free to
declare tools; the provider/model determines how it responds.

------------------------------------------------------------------------

# 16. Saved System Prompts and Filter Profiles

Saved System Prompts and Moderari Filter Profiles are configuration-time
user conveniences.

Saved System Prompt names must be unique. Historical saved-prompt
versions are not required for replay because the actual resolved/effective
System Prompt content used by the materialised Filter configuration is
execution truth and may be retained as generic execution metadata.

Likewise, Filter Profiles do not require historical versions. A profile
may be edited or deleted after use because the actual resolved Filters,
implementation versions and effective configuration are retained as
execution truth in the materialised session metadata.

Neither the saved prompt name nor Filter Profile name is required to
reconstruct a historical execution.

------------------------------------------------------------------------

# 17. Session Configuration Before First Ask

Rogare must create the Pontis session before selecting a model.

The session ID is then the common identity against which the responsible
services attach their configuration:

``` text
session_id
    │
    ├── Pontis
    │     session lifecycle
    │     tools / routing
    │
    ├── Praebere
    │     provider/model reservation
    │
    └── Moderari
          Filter Profile
          resolved effective Filters
```

The normal configuration lifecycle is:

``` text
Pontis
  │
  └── create session
          │
          ▼
       session_id
          │
          ▼
Praebere
  │
  └── reserve selected provider/model for session
          │
          ├── resolve Effective Model Definition
          └── obtain Filter Profile applicability from Moderari
          │
          ▼
Rogare
  │
  └── select applicable Filter Profile
          │
          ▼
Moderari
  │
  └── configure Filter Profile for existing session
          │
          ▼
       READY FOR FIRST ASK
```

There is therefore no atomic pre-execution specification that is
committed to create the session. The session already exists while its
execution configuration is assembled.

The architectural boundary is the **first Ask**. Before the first Ask
the session is being configured. On the first Ask, Pontis asks Praebere
to activate the session's reserved execution target and waits for
successful readiness confirmation before forwarding the interaction to
Moderari.

A session must not enter normal execution with an invalid or incomplete
required configuration.

------------------------------------------------------------------------

# 17A. Moderari Session State at First Ask

The first Ask is the point at which Moderari materializes the execution
environment it will use for the remainder of the session.

``` text
Moderari Session State
    │
    ├── session_id
    ├── provider/model execution information
    ├── required model characteristics
    ├── Effective Model Definition provenance/revision
    ├── resolved Filters + implementation versions/configuration
    ├── resolved Adapter + version
    └── other execution state
```

Praebere remains authoritative for model knowledge. Moderari is not
creating a second model catalogue; it is retaining the authoritative
model information required by this configured execution session.

The state remains fixed for the session lifecycle and provides the
baseline execution environment required for Vestigare and Repetere.

------------------------------------------------------------------------

# 18. Failure and Race Conditions

Model availability and model knowledge can change between UI selection
and the first Ask.

Therefore Rogare's display-time validation is advisory.

The authoritative checks must occur again before the session crosses the
first-Ask execution boundary.

Examples include:

``` text
model unloaded
provider unavailable
model definition changed
profile version changed
filter removed
filter requirement changed
```

A failure should reject the session configuration clearly rather than
silently falling back to another model, profile, filter or
characteristic assumption.

------------------------------------------------------------------------

# 19. Praebere Requirements

## PBR-01 --- Model Authority

Praebere is the sole Lumen authority for provider/model identity,
availability and effective model characteristics.

------------------------------------------------------------------------

## PBR-02 --- Minimum Model Definition

Praebere must initially support model identity, family, context window,
system-message capability, tool capability and streaming capability.

------------------------------------------------------------------------

## PBR-03 --- Explicit Unknown

Missing model knowledge must remain explicitly unknown and must not be
converted to false or an assumed default.

------------------------------------------------------------------------

## PBR-04 --- Model Catalogue

Praebere must provide a persistent model catalogue capable of combining
Lumen-supplied definitions with researcher/user-supplied definitions.

------------------------------------------------------------------------

## PBR-05 --- User Extensibility

A researcher must be able to add model definitions for models unknown to
the Lumen-supplied catalogue without modifying Praebere code.

------------------------------------------------------------------------

## PBR-06 --- Upgrade Safety

Lumen-supplied model definitions and user-maintained definitions must be
separable so Lumen upgrades do not overwrite researcher configuration.

------------------------------------------------------------------------

## PBR-07 --- Characteristic Provenance

Praebere must preserve the source of effective model characteristics
sufficiently to distinguish provider-discovered, Lumen-catalogue,
user-supplied and explicitly inferred information.

------------------------------------------------------------------------

## PBR-08 --- Effective Model Definition

Praebere must resolve available model knowledge into a single effective
model definition consumed by Lumen services.

------------------------------------------------------------------------

## PBR-09 --- Incomplete Models

Praebere must permit available models with incomplete definitions and
expose which characteristics remain unknown.

------------------------------------------------------------------------

## PBR-10 --- No Moderari Semantics

Praebere must not contain knowledge of individual Moderari filters or
profiles. It supplies model characteristics; Moderari determines
filter/profile applicability.

------------------------------------------------------------------------

------------------------------------------------------------------------

## PBR-12 --- Single User Model Profile

Each model may have at most one Praebere User Model Profile. The profile
is a gap-filling resource and must not create a competing model
authority.

------------------------------------------------------------------------

## PBR-13 --- Automatic Knowledge First

Praebere must not require the researcher to supply information that
Praebere can reliably discover or verify automatically.

------------------------------------------------------------------------

## PBR-14 --- Availability vs Completeness

Model runtime availability and model-definition completeness are
independent states. An incomplete model definition must not by itself
prevent the model from being selected.

------------------------------------------------------------------------

## PBR-15 --- Missing Information Visibility

Praebere must identify models with incomplete definitions and expose the
specific unknown characteristics that the researcher may supply through
the model's User Model Profile.

------------------------------------------------------------------------

## PBR-16 --- Per-Session Model Reservation

Praebere must replace the single-global-model assumption with an
explicit provider/model reservation associated with each Pontis session
ID.

------------------------------------------------------------------------

## PBR-17 --- Concurrent Model Reservations

Praebere must support different active sessions reserving the same model
or different models concurrently, subject to provider/runtime
capability.

------------------------------------------------------------------------

## PBR-18 --- Session Execution Resolution

Praebere must expose an authoritative operation through which Moderari
can resolve a Pontis session ID to its current provider/model execution
target and Effective Model Definition when the Moderari Session State is
materialised on the first Ask.

Subsequent Asks use that materialised Moderari Session State and do not
repeat model-definition resolution through Praebere.

------------------------------------------------------------------------

## PBR-19 --- Execution Readiness

On first-Ask execution activation from Pontis, Praebere must resolve the
session reservation, ensure the provider/model target is ready, load the
model where required, and return success only when execution may
proceed.

------------------------------------------------------------------------

## PBR-20 --- Session Release and Model Lifecycle

When Pontis reports session end, Praebere must release that session's
provider/model reservation and retain or unload the model according to
remaining session usage, residency ownership and Praebere lifecycle
policy.

------------------------------------------------------------------------

## PBR-21 --- Session-Configuration Applicability Mediation

When Rogare selects a model for an existing Pontis session, Praebere
should supply the model's authoritative Effective Model Definition to
Moderari for Filter Profile applicability evaluation and relay
Moderari's result to Rogare.

Praebere must not calculate Filter/Profile applicability itself or
acquire Moderari-specific Filter/Profile semantics.

------------------------------------------------------------------------

# 20. Rogare Requirements

## RGR-01 --- Model-First Configuration

Rogare must allow the execution target to be selected before presenting
the final Moderari Filter Profile choice for a new session.

------------------------------------------------------------------------

## RGR-02 --- Dynamic Profile Applicability

After model selection through Praebere, Rogare must receive Moderari's
Filter Profile applicability result through the Praebere-mediated
selection flow rather than calculating compatibility itself.

------------------------------------------------------------------------

## RGR-03 --- Applicable Profile Display

Rogare must present only the Moderari Filter Profiles that Moderari reports as
applicable to the Effective Model Definition for the selected session model,
together with explicit `No Filter Profile`.

Rogare must not calculate Filter/Profile compatibility itself.

------------------------------------------------------------------------

## RGR-04 --- No Unavailable Profile Remediation

Rogare must not present unavailable Filter Profiles as selectable session
alternatives and must not direct the researcher to alter the selected model's
characteristics merely to satisfy a Filter Profile requirement.

Model knowledge and user overrides remain a separate Praebere responsibility.

------------------------------------------------------------------------

## RGR-05 --- Session-First Configuration

Rogare must create the Pontis session first, then use that session ID to
select the provider/model through Praebere and the Moderari Filter
Profile before the first Ask.

------------------------------------------------------------------------

## RGR-06 --- Configuration Before Execution

Rogare must complete the required provider/model and Moderari Filter
Profile configuration against the existing Pontis session before issuing
the first Ask.

## RGR-07 --- Filter Profile Selection Delivery

When the user makes the Moderari Filter Profile decision for an existing
Pontis session, Rogare must send the selection and `session_id` to
Moderari through Nuntius using `\obt`. The selection may be a named
Filter Profile or explicit `None` for No Filter Profile. Filter Profile
selection must not be deferred until the first Ask or embedded in the
ordinary Ask payload.

## RGR-08 --- Interaction Readiness

Starting a session creates the Pontis session but does not make Rogare
ready for interaction. Rogare must keep the User Context prompt input
and Ask/send controls disabled until the session ID exists, model
selection is complete, and the Filter Profile decision is complete.

The Filter Profile control becomes actionable only after model
selection, because Filter Profile applicability depends on the selected
model's Effective Model Definition. Explicit No Filter Profile (`None`)
satisfies the Filter Profile decision requirement.

``` text
Start Session
      │
      ▼
Session Created
      │
      │ select model
      ▼
Model Selected
      │
      │ select Filter Profile
      │ or No Filter Profile
      ▼
Session Configured
      │
      ├── User Context enabled
      └── Ask/send enabled
```

Rogare must distinguish:

``` text
UNSELECTED    no Filter Profile decision / not ready
None          explicit No Filter Profile / ready
<profile-id>  named Filter Profile / ready
```

------------------------------------------------------------------------

## RGR-10 --- Authoritative Revalidation

Rogare's UI state is advisory. The execution target and Moderari Filter
Profile must be authoritatively revalidated when the session
specification is committed.

------------------------------------------------------------------------

## RGR-11 --- No Silent Fallback

Rogare must not silently substitute another model, profile, filter or
assumed capability when the selected session configuration cannot be
satisfied.

------------------------------------------------------------------------

## RGR-09 --- Filter Administration Boundary

Rogare is a session client, not the Moderari Filter administration UI.
It may list selectable Filter Profiles and, where useful, available
Filter identifiers. Detailed Filter descriptions, configuration schemas,
and creation/editing of Filter Profiles belong to the Moderari UI in
Servire.

------------------------------------------------------------------------

# 21. Pontis Changes and Requirements

The redesign requires Pontis changes because the current implementation
still reflects the single-global-model architecture.

Pontis should continue to own:

``` text
session creation and lifecycle
client request/response transport/routing
tool setup and tool execution integration
forwarding of client \obt and other control requests
first-Ask execution activation gate
session-end notification
```

Pontis should **not** become an AI interaction-representation classifier.
Ordinary client requests are forwarded to Moderari with their session identity;
Moderari's Adapter Framework determines the client representation.

Pontis should no longer be the authority for a global or per-session
selected model. The provider/model reservation belongs to Praebere.

## PTR-01 --- Session Authority

Pontis must remain authoritative for the Lumen session ID and session
lifecycle, without becoming the authority for provider/model knowledge.

------------------------------------------------------------------------

## PTR-01A --- Session Before Model Selection

Pontis must create the session and provide its session ID before a
provider/model can be selected or reserved for that session.

------------------------------------------------------------------------

## PTR-02 --- Praebere Model Reservation

When a model is selected for a Pontis session, Pontis must cause
Praebere to establish or update the provider/model reservation for that
session.

------------------------------------------------------------------------

## PTR-03 --- First-Ask Activation Gate

Before forwarding the first ordinary Ask for a session to Moderari,
Pontis must activate the execution session through Praebere and wait for
successful confirmation that the session's reserved execution target is
ready.

If activation fails, Pontis must not forward the Ask to Moderari.

------------------------------------------------------------------------

## PTR-04 --- No Repeated Activation While Active

Pontis should not repeat Praebere execution activation for every Ask
while the execution session remains active.

------------------------------------------------------------------------

## PTR-05 --- Remove Global Model Authority

Pontis must not maintain a global authoritative model as part of the new
multi-session architecture. Any current global authoritative-model state
and request rewriting based upon it must be removed or replaced by
session-aware routing that relies on Praebere's reservation authority.

------------------------------------------------------------------------

## PTR-06 --- Session End Notification

When a Pontis session ends, Pontis must notify services holding
session-scoped state.

Praebere must be notified so it can release the session's provider/model
reservation and apply its ownership and unload rules.

Moderari must be notified so it can remove all live state associated
with the `session_id`, whether that state is only an unused
pre-execution Filter Profile selection or a fully materialized Moderari
Session State.

------------------------------------------------------------------------

## PTR-07 --- No Model Lifecycle Decisions

Pontis must not decide whether a provider/model should be loaded,
retained or unloaded. Those decisions belong to Praebere.

------------------------------------------------------------------------

## PTR-08 --- Moderari Handoff

After successful first-Ask activation, Pontis records that the session
is execution-active and forwards the interaction and session ID to
Moderari. Pontis does not classify or supply the client's AI interaction
representation, does not supply Moderari with a cached model definition,
and does not retain the provider/model associated with the activation.

On the first Ask, Moderari obtains the authoritative Effective Model
Definition and provider/model execution information directly from
Praebere and materializes its immutable Moderari Session State. On
subsequent Asks, Moderari uses that established session state without
repeating model-definition resolution through Praebere.

------------------------------------------------------------------------

## PTR-09 --- No Client Representation Classification

Pontis must not require representation-specific knowledge in order to route an
ordinary interaction to Moderari. Recognition of OpenAI, Anthropic or future
AI interaction representations belongs to the Moderari Adapter Framework.

------------------------------------------------------------------------

# 21A. Pre-Execution Moderari Configuration and Cleanup

A Moderari Filter Profile decision belongs to an existing Pontis session
before execution begins. The user may select a named Filter Profile or
explicitly select **No Filter Profile**. Rogare sends that decision
immediately to Moderari through Nuntius:

``` text
Rogare
   │
   │ \obt moderari profile select
   │ session_id=<session_id>
   │ profile_id=<profile>
   ▼
Nuntius
   │
   ▼
Moderari
   │
   └── session_id -> selected Filter Profile
       (pre-execution session configuration)
```

For No Filter Profile the value is explicitly `None`. This is a valid
configured state resolving to zero intentional Filters. It is distinct
from `UNSELECTED`, meaning no Filter Profile decision has yet been made.

The Filter Profile is therefore not carried in the ordinary first-Ask
payload. Pontis remains unaware of the Moderari configuration.

On the first Ask, Moderari obtains the authoritative Effective Model
Definition and provider/model execution information from Praebere and
combines them with the Filter Profile already associated with the
`session_id` to materialize the immutable Moderari Session State.

Pontis remains the authority for session lifetime. On session end,
Pontis notifies both Praebere and Moderari. Praebere releases the
provider/model reservation; Moderari removes all live state for the
session.

This cleanup also applies when the session never executed. If a Filter
Profile was selected and the user ends the session before the first Ask,
Moderari removes the unused pre-execution association.

Once execution has begun, the materialized Moderari Session State is
locked. Selecting a different Filter Profile requires a new session.

Historical evidence is independent of this live-state cleanup. If the
session executed, Vestigare retains whatever evidence its recording
policy caused it to persist.

------------------------------------------------------------------------

# 22. Vestigare Changes --- Raw Trace and Execution Metadata

Vestigare preserves the true Client ↔ Lumen interaction independently of internal Moderari Filter operations.

> **Vestigare records what was said and the generic execution environment under which it was said.**

For each recorded interaction the raw trace preserves:

``` text
exact client interaction/context presented to Lumen
exact response returned to the client in the client interaction representation boundary
session/exchange correlation
ordinary trace metadata
```

Filter-transformed context is not written into the Vestigare conversation trace. Adapters/CIM do not create a second conversation.

## 22.1 Raw Trace Invariant

The raw trace must remain usable independently of the Moderari Filter environment that happened to be active during the original execution.

## 22.2 Execution Metadata

When Moderari materialises its Session State, it sends the relevant execution environment metadata to Vestigare using the existing execution-metadata facility.

This may include provider/model execution identity, Effective Model Definition provenance/revision, client/provider interaction representations, resolved Adapter implementation/state/version, resolved Filter identities and implementation versions, effective Filter configuration, and other relevant materialised Moderari session state.

This extends execution information Vestigare already records, such as the model used. It does not become raw conversation content.

## 22.3 Vestigare Requirements

### VTR-01 --- Exact Client Interaction

Vestigare must preserve the exact client interaction/context presented at the Lumen boundary.

### VTR-02 --- Exact Client-Visible Response

Vestigare must preserve the exact response returned to the client in the client interaction representation.

### VTR-03 --- Filter Independence

Vestigare must not store Filter-transformed context as part of the raw conversation trace and must not require Filter-specific transformation schemas.

### VTR-04 --- Adapter Representation Independence

Adapter/CIM representation changes must not create alternate conversation content in Vestigare.

### VTR-05 --- Correlation

The raw trace and execution metadata must carry sufficient session/exchange identity to be correlated with any separately retained Moderari Filter transformation records.

### VTR-06 --- Generic Execution Metadata

Vestigare must be able to retain the relevant materialised Moderari session configuration as generic execution metadata without treating it as raw conversation content.

------------------------------------------------------------------------

# 23. Moderari Filter Transformation Evidence

Moderari owns records describing what its Filters actually did during interaction execution.

These may include session/exchange correlation identity, Filter ID/version, execution/application records and transformation/intermediate-state records.

These records are separate from both the Vestigare raw trace and Vestigare generic execution metadata. Vestigare does not need to understand or store those intermediate states.

Aestimare or external research tooling may correlate the Vestigare raw trace, Vestigare execution metadata and Moderari Filter transformation evidence through shared session/exchange identities.

# 24. Repetere Changes --- Current Scope

Repetere uses the Vestigare client-boundary conversation as its replay source material. The conversation is independent of the Moderari Filter transformations that occurred during the original execution.

The materialised Moderari Session Execution State is retained separately as Vestigare execution metadata and describes the execution environment under which that conversation originally ran.

Repetere therefore supports two conceptually distinct replay setups:

``` text
Vestigare
│
├── Client-Boundary Interaction Trace
│
└── Recorded Moderari Session Execution State
        │
        ▼
     Repetere
        │
        ├── Original Setup
        │      └── reconstruct recorded Session Execution State
        │
        └── New Setup
               ├── select Model
               └── select Moderari Filter Profile / None
                        │
                        ▼
                 normal Lumen session configuration
                        │
                        ├── Praebere resolves model
                        ├── Moderari resolves Filters
                        └── Moderari resolves Adapter
                        │
                        ▼
                 new Session Execution State
```

For an **Original Setup** replay, Repetere obtains the recorded Moderari Session Execution State and requests that Moderari establish that recorded execution environment before submitting the first recorded Ask. That state is self-contained execution truth: it contains the actual resolved Filter implementations and versions and their complete effective configurations. For the System Prompt Filter, this includes the actual System Prompt text used during the original execution.

Moderari determines whether the recorded execution path can still be constructed from the execution components currently available. If a required recorded Filter implementation/version, Adapter implementation/version, or other required execution component is unavailable, Moderari rejects the Original Setup. It must not silently substitute a newer or different implementation.

Repetere therefore does not independently reconstruct Filter or Adapter execution behaviour. If Moderari rejects the Original Setup, the user may create a **New Setup** through the normal Lumen session-configuration path.

Original Setup reconstruction does not depend on the continued existence of the original Filter Profile, saved System Prompt name, or other mutable configuration-time resources. It does depend on the required recorded executable implementations still being available.

For a **New Setup** replay, Repetere does not modify or selectively override the recorded Session Execution State. Instead, a new replay session is configured through the normal Lumen configuration path by selecting a model and a Moderari Filter Profile, including explicit `None`. Praebere and Moderari then resolve the new execution environment normally, including Filter applicability, effective Filters and Adapter selection.

This is important because changing the replay model may itself change which Filters are applicable or which Adapter is required. Those dependent execution decisions are resolved afresh rather than inherited from or patched into the original Session Execution State.

`No Filters` is therefore not a separate replay mechanism. It is a **New Setup** with an explicit `None` Filter Profile decision. Likewise, Repetere does not select a particular Adapter as an independent replay override; Adapter resolution remains a Moderari responsibility derived from the newly configured execution target and client interaction representation.

In both replay modes, the recorded client-boundary interaction remains unchanged. A replay is a new execution and materialises its own Session Execution State and evidence.

> **Replay source material is the recorded client-boundary conversation. A replay either reconstructs the original recorded Session Execution State or creates a new Session Execution State through normal Lumen session configuration. Repetere does not mutate the recorded execution state.**

## Repetere Requirements

### RPR-01 --- Client-Boundary Trace Source

Repetere must treat the Vestigare client-boundary conversation as replay source material.

### RPR-02 --- No Embedded Filter State

Repetere must not depend on Filter-transformed model-facing context being embedded in the Vestigare conversation trace.

### RPR-03 --- Original Setup Reconstruction Request

For an Original Setup replay, Repetere must obtain the recorded, self-contained Moderari Session Execution State and request that Moderari establish that execution environment before submitting the first replay Ask. Repetere must not independently reconstruct Filter or Adapter execution behaviour.

### RPR-04 --- New Execution

A replay is a new execution and produces its own client-boundary trace, Session Execution State and associated execution evidence.

### RPR-05 --- New Setup Uses Normal Configuration

For a New Setup replay, Repetere must establish a new session through the normal Lumen configuration path using a selected model and Moderari Filter Profile or explicit `None`. It must not mutate individual fields of the recorded Session Execution State.

### RPR-06 --- Derived Execution Decisions Are Re-resolved

For a New Setup replay, Praebere and Moderari must resolve the new execution environment normally. Adapter selection, Filter applicability and effective Filter configuration must therefore be derived from the new session configuration rather than inherited from the recorded execution state.

### RPR-07 --- Replay Independence from Mutable User Configuration

An Original Setup replay must not depend on current Filter Profile definitions, saved System Prompt names/content, or other mutable user-convenience configuration. The recorded Session Execution State is the replay authority for the original execution environment.

### RPR-08 --- Original Setup Availability

If Moderari reports that the recorded execution environment cannot be reconstructed because a required recorded Filter, Adapter, implementation version, or other required execution component is unavailable, Repetere must reject the Original Setup replay. Repetere must not request or perform substitution with a newer or different implementation.

### RPR-09 --- New Setup After Reconstruction Failure

Failure to reconstruct an Original Setup does not prevent replay of the recorded client-boundary interaction. The user may create a New Setup through the normal Lumen session-configuration path using currently available Models, Filters and Adapters.

------------------------------------------------------------------------

# 24A. Servire Handling of Invalid Moderari Extension Configuration

Moderari extension discovery is part of establishing a valid Stack runtime
configuration.

If Moderari detects duplicate Adapter Registry keys during startup, Moderari
must fail startup rather than choose between competing executable extensions.

``` text
Stack startup
    │
    ▼
Moderari discovers Adapters
    │
    ▼
Build Adapter Registry
    │
    ├── valid unique keys -> Moderari startup continues
    │
    └── duplicate provider + model key
             │
             ▼
       Moderari startup failure
             │
             ▼
           Servire
             │
             ▼
       Stack rollback
```

The responsibility boundary is:

``` text
Moderari
    detect duplicate registry key
    identify conflicting Adapters and versions
    report deterministic startup failure

Servire
    observe Moderari startup failure
    treat Stack startup as failed
    invoke the normal Stack rollback mechanism
    surface the failure to the operator
```

Servire must not attempt to choose, rank, disable or otherwise resolve the
conflicting Adapters on Moderari's behalf. The installed extension
configuration must be corrected before the Stack can start successfully.

## SVR-01 --- Moderari Startup Failure Causes Stack Rollback

When Moderari fails startup because its Adapter Registry cannot be constructed
unambiguously, Servire must treat that as a Stack startup failure and invoke
its normal Stack rollback mechanism.

## SVR-02 --- Extension Conflict Is Not Auto-Resolved

Servire must preserve Moderari's duplicate-Adapter failure rather than
automatically selecting, disabling or preferring one of the conflicting
extensions.

------------------------------------------------------------------------

# 25. Fiducia Follow-On

Future Repetere work may introduce deliberate variation of Models,
Filters, System Prompts or Adapter state. If that happens, Fiducia's
scheduled-replay contract may also require extension.

That work is explicitly outside the current scope and should not
constrain the present Moderari/Praebere/Pontis/Rogare/Vestigare changes.

------------------------------------------------------------------------

# 21. Companion Acceptance Criteria

The Praebere changes satisfy the architecture when:

> **A provider can expose a previously unknown model, Praebere can
> represent its known and unknown characteristics, a researcher can
> supplement its definition without modifying code, and Lumen can obtain
> one provenance-aware effective model definition from Praebere.**

The Rogare changes satisfy the architecture when:

> **Rogare can create the Pontis session, select a model for that
> existing session, present Moderari's applicable Filter Profiles,
> require an explicit Filter Profile or No Filter Profile decision,
> retain and send its own conversational context, and use a Filter
> Profile for any Lumen/user System Prompt without duplicating model-,
> Adapter- or Filter-specific compatibility logic.**

Together with the Moderari acceptance criterion, the complete design
objective is:

``` text
Pontis
    owns the session and routing
        │
        ▼
Praebere
    owns session -> provider/model reservation
    and model truth/lifecycle
        │
        ▼
Moderari
    materializes model/provider truth on first Ask
    and owns Adapter/Filter transformation requirements
        │
        ▼
Rogare
    presents valid choices
```

This keeps the services independently evolvable while making the user's
session configuration safer, clearer and reproducible.

Detailed extension behaviour is defined separately in:

``` text
Lumen Moderari Adapter Architecture
Lumen Moderari Filter Architecture
```


# 26. 02026-10-02 Companion Clarifications — Vestigare and Repetere

## 26.1 Vestigare Records the Client Boundary

Vestigare conversation evidence is defined at the Client ↔ Lumen boundary. It records the Ask actually received from the client and the response actually returned to the client, in the client interaction representation.

A provider-side representation is an execution intermediate. If an Adapter translates the provider response into the client representation, Vestigare records the translated client-facing response. If no Adapter is required, the response already has the client representation.

Neither Adapter extensions nor Filter extensions may directly modify Vestigare raw trace records.

## 26.2 Session Execution Metadata Is Replay Authority

The materialised Moderari Session Execution State stored as Vestigare execution metadata is the authoritative description of what Moderari actually used. It must contain the resolved Filter identities, implementation versions and complete effective configurations, together with the effective Adapter implementation/state/version and other required execution facts.

For the System Prompt Filter, the execution state contains the **actual System Prompt content used**. It must not rely on a saved System Prompt name, Filter Profile identity, or other mutable configuration resource. Filter Profiles and saved prompts remain user-convenience configuration resources only.

## 26.3 Repetere Replay Setup

For an Original Setup replay, Repetere begins by parsing the recorded Session Execution State and asks Moderari to establish the recorded execution environment from that self-contained state. Moderari validates whether the required recorded execution components are still available. Repetere submits the original first recorded Ask only after Moderari has successfully established the execution path.

``` text
Vestigare Session Execution Metadata
        │
        ▼
Repetere parses execution state
        │
        ▼
request recorded execution path
        │
        ▼
Moderari validates required
Filters / Adapters / versions
        │
        ├── available ─────► configure replay session
        │                         │
        │                         ▼
        │                  Original first Ask
        │                         │
        │                         ▼
        │                  normal Filter + Adapter execution
        │
        └── unavailable ───► reject Original Setup
                                  │
                                  ▼
                           user may create
                              New Setup
```

Moderari must not silently substitute a newer or different Filter or Adapter implementation when establishing an Original Setup. If the recorded execution path is unavailable, Repetere reports the Original Setup as unavailable and leaves creation of a New Setup to the normal Lumen configuration path.

If the original Ask contained a System Prompt, it remains part of the original client interaction. The reconstructed System Prompt Filter applies the recorded effective replacement/configuration during replay.

For a New Setup replay, Repetere does not reconstruct and then override portions of the original execution state. It creates a new replay session through normal Lumen configuration by selecting a model and Moderari Filter Profile or explicit `None`. Praebere and Moderari then resolve the resulting execution environment, including the effective Adapter and Filters, and materialise a new Session Execution State before the replay Ask executes.

The existing special System Prompt replacement backchannel to Vestigare and the replay-time System Prompt pass-through behaviour are legacy mechanisms superseded by execution-state reconstruction and should be removed.

## 26.4 Cognition Evidence Boundary

Cognition private inference is an internal Moderari operation and does not create a Vestigare conversation-trace exchange. Its configured Filter identity/version/effective configuration forms part of Session Execution State. Any ordinary model response subsequently returned through Lumen to the client is recorded normally at the client boundary.

### VTR-07 — Extension Trace Immutability

Adapters and Filters must have no extension API capable of adding, removing, replacing or suppressing Vestigare raw conversation records.

