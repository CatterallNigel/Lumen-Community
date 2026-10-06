# Lumen Moderari Redesign

## Responsibilities, Extension Boundaries and Session Architecture

## 1. Purpose

The Moderari redesign should begin by separating what Moderari
**fundamentally needs to do** from functionality that accumulated as
Lumen evolved.

Moderari was originally developed heavily around Qwen as a locally
runnable model, Ollama as the provider, Pi as the principal client/tool
environment, and the need to translate Qwen's representation of tool
calls into the OpenAI-compatible protocol expected by the surrounding
system.

Lumen has since developed substantially.

It now has distinct services with clearer areas of authority, including
Praebere, Pontis, Vestigare, Repetere, Nuntius, Rogare, Fiducia and
Aestimare.

The objective is therefore not simply to refactor the current Moderari
implementation.

The starting question is:

> **If Moderari were designed today for the current Lumen architecture,
> what is the smallest service required in that position?**

Capabilities can then be added deliberately on top of that minimal
service.

------------------------------------------------------------------------

# 1. Vanilla Moderari

The fundamental Moderari path is:

``` text
Pontis
   │
   ▼
Moderari
   │
   ▼
Provider / Model
   │
   ▼
Moderari
   │
   ▼
Pontis
```

Vanilla Moderari should perform as little intervention as possible.

Its essential role is to provide the boundary between Lumen's
interaction and the selected provider/model execution target.

A useful principle is:

> **Vanilla Moderari does not modify the semantic content of an
> interaction.**

It may perform technical adaptation when an applicable Adapter path is enabled and required to communicate correctly with the selected provider/model, but it does not otherwise
attempt to improve, steer or modify the interaction.

An empty filter set is therefore a valid and important Moderari
configuration.

``` text
Client interaction
       │
       ▼
    Moderari
       │
       │ no semantic modification
       ▼
Technical adapter
if required
       │
       ▼
Provider / Model
```

This provides Lumen with a clean experimental baseline.

------------------------------------------------------------------------

# 2. Praebere Owns Provider and Model Knowledge

Analysis of the existing Qwen profile shows that it currently mixes
several unrelated concepts:

-   model identity;
-   context characteristics;
-   sampling configuration;
-   Qwen behavioural corrections;
-   Pi tool instructions;
-   Ollama-specific configuration;
-   batch workflow rules;
-   compaction policy;
-   system-prompt policy;
-   timeout policy.

These responsibilities should be separated.

The new architectural principle is:

> **Anything describing the provider, model or model characteristics
> belongs to Praebere.**

Conceptually:

``` text
Praebere
│
├── Provider
│   ├── identity
│   ├── endpoint
│   ├── protocol
│   └── capabilities
│
└── Model
    ├── identity
    ├── family
    ├── context characteristics
    ├── tool capability
    ├── system-message capability
    ├── sampling capabilities
    └── other discovered/known characteristics
```

How much of this information Praebere can obtain automatically from a
provider remains an open question.

Praebere may ultimately combine:

``` text
Provider discovery
       +
Lumen-known model metadata
       +
explicit configuration where necessary
```

The important point is that **Praebere remains the authority**.

Moderari should not independently maintain competing model knowledge or
infer model characteristics from model-name matching.

------------------------------------------------------------------------

# 3. Moderari Session Configuration and Materialisation

Moderari participates in session configuration before the first Ask. The provider/model reservation and Filter decision are therefore established before ordinary interaction begins. The first Ask is not used to rediscover model knowledge already supplied for the session; it supplies the request-boundary information needed to recognise the client interaction representation and resolve the effective Adapter path.

The session lifecycle is:

``` text
SESSION CREATED
    │
    │ Pontis owns session_id
    ▼
MODEL SELECTED
    │
    │ Praebere reserves provider/model for session
    │ and supplies authoritative execution/model information
    ▼
FILTER PROFILE APPLICABILITY
    │
    │ Moderari evaluates valid profiles against
    │ the Effective Model Definition
    ▼
FILTER PROFILE DECISION
    │
    │ named profile OR explicit None
    ▼
SESSION CONFIGURED / INTERACTION-READY
    │
    │ Moderari owns session-scoped execution configuration
    ▼
FIRST ASK
    │
    ├── recognise client interaction representation
    ├── resolve effective Adapter path
    └── validate executable path
    ▼
EXECUTION PATH ESTABLISHED
    │
    ▼
SUBSEQUENT ASKS
    │
    ▼
SESSION END / CLEANUP
```

## 3.1 Session Creation

Pontis creates the Lumen session and owns its lifetime.

``` text
Rogare / Client
      │
      ▼
    Pontis
      │
      └── create session_id
```

A newly created session is not yet interaction-ready.

## 3.2 Model Selection and Reservation

After the Pontis session exists, Rogare selects the model through Praebere using that `session_id`.

Praebere owns the resulting reservation:

``` text
session_id -> provider -> model
```

The reservation remains in force until Pontis ends the session. Praebere resolves the Effective Model Definition, provider interaction representation and provider/model execution information for that reservation and supplies the required authoritative information to Moderari.

Moderari may retain those facts as session-scoped execution configuration. This does not create a competing model catalogue: Praebere remains the persistent authority for provider/model knowledge; Moderari owns only the execution configuration established for the session.

## 3.3 Filter Profile Applicability

Once Praebere has established the model reservation, Moderari first determines which current Filter Profiles are **valid**. A profile is valid only when every Filter it references is currently available. A profile containing a missing Filter remains stored and editable in the Moderari UI in Servire, but it is invalid for execution and must not be offered to Rogare or another client for selection.

Moderari then evaluates model applicability for valid profiles against the authoritative Effective Model Definition supplied for the session.

``` text
Praebere
    reserve provider/model
    + Effective Model Definition
             │
             ▼
Moderari
    retain session execution facts
    evaluate Filter requirements
             │
             ▼
Rogare
    present applicable profiles
```

At this configuration stage the client interaction representation may not yet be known. Therefore the returned profiles are **model-applicable candidates**, not a guarantee that a selected profile can execute. Client-representation compatibility is validated when Moderari receives the first Ask.

Praebere supplies model truth; Moderari owns Filter Profile validity and applicability; Rogare presents the result supplied by Moderari.

## 3.4 Filter Profile Selection

The user must make an explicit Filter Profile decision before interaction begins. The decision is either a profile identifier or explicit `None`.

`None` means **No Filter Profile** and resolves to zero intentional Filters. It is a completed configuration decision and is distinct from `UNSELECTED`.

Rogare sends the selection and `session_id` to Moderari through Nuntius using `\obt`. Moderari resolves the selected profile to its effective Filter set and configuration and adds that decision to the session-scoped execution configuration.

``` text
Rogare
  │
  │ \obt moderari profile select
  │ session_id=<session_id>
  │ profile_id=<profile-or-none>
  ▼
Nuntius
  │
  ▼
Moderari
  │
  ├── session provider/model facts already known
  ├── resolve Profile -> effective Filters/configuration
  └── session configuration fixed
```

The Filter Profile remains configuration-time intent. The resolved Filter set and complete effective configuration are execution-time truth.

## 3.5 Interaction Readiness

The session becomes interaction-ready only when:

``` text
Pontis session exists
+
Praebere provider/model reservation exists
+
Moderari Filter Profile decision exists
```

Rogare keeps its interaction controls disabled until all three conditions are satisfied.

> **Start Session creates a session; it does not make the session interaction-ready.**

Interaction-ready means that all configuration that can be established before seeing an ordinary client request has been established. It does not imply that Moderari has yet recognised the client interaction representation or resolved the effective Adapter path.

## 3.6 First-Ask Execution Activation

Before the first ordinary Ask is forwarded to Moderari, Pontis activates the execution session through Praebere.

Praebere uses the existing session reservation, ensures provider/model readiness, loads the model where required, and confirms successful activation. This is an operational readiness gate; it is not a second model-selection or model-definition lookup for Moderari.

``` text
FIRST ASK — EXECUTION ACTIVATION

Client
  │
  ▼
Pontis
  │
  │ activate execution(session_id)
  ▼
Praebere
  │
  ├── use existing session reservation
  ├── ensure provider/model readiness
  ├── load model if required
  └── confirm execution ready
          │
          ▼
        Pontis
          │
          ├── mark session execution active
          └── forward first Ask
```

Pontis remembers that activation happened. Praebere remembers what was activated.

## 3.7 First-Ask Client Representation and Execution-Path Resolution

On the first Ask, Moderari uses the request boundary to attempt to recognise the client interaction representation. The provider interaction representation is already known from the Praebere session reservation.

Moderari then resolves whether the configured session has an executable representation path:

``` text
FIRST ASK

recognise client representation
        │
        ├── Unknown
        │      │
        │      ├── No Filters ─────────────► PASS THROUGH
        │      │
        │      └── Filters selected ───────► REJECT FIRST ASK
        │
        └── Known
               │
               ▼
        resolve client ↔ provider path
               │
               ├── compatible ─────────────► EXECUTE
               └── incompatible ───────────► REJECT FIRST ASK
```

The authoritative rule is:

| Situation                                                                                  | Moderari action  |
| ------------------------------------------------------------------------------------------ | ---------------- |
| Client representation `Unknown` + No Filters                                               | Pass through     |
| Client representation `Unknown` + Filters selected                                         | Reject first Ask |
| Client and provider representations known but incompatible, but no compatible Adapter path | Reject first Ask |
| Client and provider representations known + compatible execution path                      | Execute          |

An Unknown client representation is therefore not inherently an execution failure. With no intentional transformation selected, Vanilla Moderari does not interfere and the original interaction may be passed to the provider. Once Filters have been selected, however, Moderari must positively establish the client representation because it must be able to materialise Final Filter Output in that representation.

Likewise, when both representations are known, but incompatible and Moderari can positively establish that no compatible execution path exists, it rejects the Ask rather than knowingly forwarding an incompatible representation.

The effective Adapter path resolved here becomes part of the session execution state and is reused for subsequent Asks.

## 3.8 Session Execution State

After successful first-Ask representation/path resolution, Moderari's session state consists of the configuration already fixed before the Ask plus the client representation and effective Adapter path established from the first request.

Conceptually:

``` text
Moderari Session Execution State
│
├── session_id
├── authoritative provider/model execution information
├── Effective Model Definition/provenance required by Moderari
├── resolved Filters + complete configuration
├── client interaction representation
├── effective Adapter implementation/state/path
└── other execution state
```

The resulting state is the immutable execution baseline for the session.

Moderari sends the relevant execution-state metadata to Vestigare using the existing execution-metadata mechanism (if a backchannel - replace with \obt). For each resolved Filter this includes identity, implementation version and complete effective configuration actually used. For the System Prompt Filter, this includes the actual System Prompt text used for execution, not a saved System Prompt name or Filter Profile reference.

Changes to Praebere model knowledge, Filter Profile definitions, installed Filters or installed Adapters must not silently alter an executing session.

## 3.9 Subsequent Asks

Subsequent Asks use the established session state:

``` text
Client
  │
  ▼
Pontis
  │
  ├── session already execution-active
  └── no repeated Praebere activation
          │
          ▼
        Moderari
          │
          ├── retrieve Session Execution State
          ├── use fixed provider/model information
          ├── use resolved Adapter path/state
          ├── use resolved Filter configuration
          └── execute interaction
```

## 3.10 Session-End Cleanup

Pontis owns the lifetime of the Lumen session. On session end it notifies services holding session-scoped state, including Praebere and Moderari.

Praebere releases the provider/model reservation according to its model-lifecycle rules. Moderari removes its live session configuration and execution state.

Historical records are independent of live-state cleanup: Vestigare retains the raw trace according to its recording policy, while Moderari retains its own session/filter execution records according to the Moderari evidence policy.

The governing principles are:

> **Praebere remains the sole persistent authority for provider/model knowledge.**

> **Moderari establishes session execution configuration as soon as authoritative information becomes available; it does not defer known configuration to the first Ask.**

> **The first Ask establishes request-boundary execution facts that could not be known during configuration, principally the client interaction representation and resulting Adapter path.**

> **Pontis owns session lifetime. Any service holding session-scoped live state releases that state when Pontis announces session end.**

# 4. Extension Architecture Overview

Moderari has two deliberately different executable extension mechanisms:

``` text
Moderari Core
│
├── Adapter Framework
│   └── technical interaction-representation compatibility
│
└── Filter Framework
    └── explicit intentional interaction transformation
```

This redesign document defines only the service-level boundary.

Adapters are selected automatically by Moderari when technical compatibility
translation is required. Their detailed registry, resolution, specialisation,
loopback and CIM contracts are defined in **Lumen Moderari Adapter
Architecture** and **Lumen Moderari Canonical Interaction Model Architecture**.

Filters are explicit session-selected intentional transformations. A Filter
Profile is configuration-time convenience which resolves to the effective
Filter set/configuration for a session. Detailed Filter execution,
applicability, persistence and extension contracts are defined in **Lumen
Moderari Filter Architecture**.

Base Representation Adapters are Lumen-controlled. Third-party Adapter
extensions are Specialised Adapters only. Third-party Filters are supported
through the Filter extension boundary. Extension discovery is restart-time for
the current research distribution, and loading third-party in-process code is
a trust decision.

For the current redesign, Moderari retains the session conversation and
model-facing context required by the existing interaction and Filter
machinery.

Moderari has no default System Prompt. Any Lumen/user System Prompt is supplied
through the System Prompt Filter. `No Filter Profile` is an explicit,
first-class session decision and resolves to zero intentional Filters.

The current Lumen-supplied Filters are:

``` text
System Prompt Filter
Cognition Filter
```

The Cognition Filter's detailed behaviour remains subject to inspection of the
existing context/compaction implementation.

> **Adapter = compatibility. Filter = intentional transformation. Filter
> Profile = user configuration convenience.**

# 19. Vestigare Raw Trace, Execution Metadata and Moderari Transformation Evidence

The evidence boundary distinguishes the raw interaction from the environment under which it occurred and from internal Filter transformations.

> **Vestigare records what was said and the execution environment under which it was said. Moderari records what its Filters did.**

Vestigare does not store Filter-transformed context as part of the raw conversation trace. For each recorded interaction, the raw trace preserves the exact client interaction/context presented to Lumen and the exact response returned to the client in the client interaction representation.

## 19.1 Vestigare Execution Metadata

When the Moderari Session State is materialised, Moderari sends the relevant session execution metadata to Vestigare using the existing execution-metadata mechanism.

Relevant metadata may include:

``` text
session_id
provider/model execution identity
Effective Model Definition provenance/revision
client/provider interaction representations
resolved Adapter implementation/state/version
resolved Filter identities
Filter implementation versions
effective Filter configuration
other relevant Moderari session execution state
```

This metadata describes **the environment under which the raw trace occurred**. It does not become conversation content. Vestigare remains Filter-agnostic and does not need to understand the semantic purpose of an installed Filter.

## 19.2 Moderari Filter Execution Evidence

Moderari owns Filter execution identity and evidence. Each Filter invocation is assigned a unique `filter_execution_id` by the Moderari Filter Framework. The Filter itself does not create that identity.

A Filter first produces material in its supported output representation. After any required Adapter Framework loopback processing, the Filter Framework obtains the **Final Filter Output** in the recognised client interaction representation. This Final Filter Output is what Moderari persists and incorporates into session context. Transient Filter-produced representations, CIM instances and loopback translation stages are implementation mechanics and are not separately persisted as the Filter output.

Where a Filter performs private model inference, as Cognition currently does, Moderari also persists the private model Ask and returned model Response as Filter execution artefacts associated with the same `filter_execution_id`. The returned private model Response is persisted after normal provider-to-client representation processing and is therefore in the client interaction representation.

Moderari knows whether material originated from the client, a Filter or a model from the execution path that produced it. Origin is not inferred from message content, role or interaction representation and is not embedded into the external representation.

``` text
Vestigare
├── Raw Interaction Trace
│     └── what was actually said at the client boundary
└── Execution Metadata
      └── environment under which it occurred

Moderari
└── Filter Execution Evidence
      └── filter_execution_id
          ├── Filter identity/version/configuration
          ├── private model Ask       # where applicable
          ├── private model Response  # where applicable
          └── Final Filter Output     # client representation
```

Aestimare or external research tooling may correlate these records using shared session/exchange and Filter execution identities.

> **Interaction representation describes what the material is. Moderari execution provenance describes where it came from.**

> **A Filter execution never becomes part of the Vestigare raw conversation trace merely because Moderari persists its execution artefacts.**

## 19.3 Filters Are To-Model Only

Moderari Filters operate only on the interaction/context presented to the model. They do not process, modify, filter, or otherwise transform the model's response. The model response is returned to the client unchanged in semantic content, subject only to any Adapter translation required to return it in the client's interaction representation.

The Cognition Filter may perform private model inference as part of constructing model-facing context. Its detailed behaviour remains deferred until the existing implementation is inspected. This private inference does not change the rule that Filters do not operate on the ordinary model response returned to the client.**

## 19.4 Adapters Are Not Filter Transformations

Adapters change representation where required for compatibility; they do not intentionally change the semantic interaction.

Where Adapter processing is Enabled, a translation path may use the CIM. Where Adapter processing is Disabled, the CIM is not introduced.

Vestigare records the raw/semantic interaction boundary, not intermediate Adapter or CIM representations. (To discuss whether Moderari persists the Adapter transformation)

# 20. Repetere Boundary for the Current Redesign

### Replay and Moderari Session Execution State

The Vestigare interaction trace is deliberately independent of the Filter transformations that occurred during the original execution.

Vestigare records the conversation at the Lumen client boundary. The materialised Moderari Session Execution State is recorded separately as execution metadata and describes the environment in which that conversation was executed.

Repetere therefore has two replay modes:

```
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
        │      │
        │      └── reconstruct recorded
        │          Session Execution State
        │
        └── New Setup
               │
               ├── select Model
               └── select Moderari Filter Profile / None
                        │
                        ▼
                 normal Lumen session
                 configuration
                        │
                        ├── Praebere resolves model
                        ├── Moderari resolves Filters
                        └── Moderari resolves Adapter
                        │
                        ▼
                 new Session Execution State
```

For an **Original Setup replay**, Repetere requests that Moderari establish the recorded Session Execution State before submitting the first recorded Ask. This state contains the actual execution configuration used by the original session, including the resolved Filter implementations and versions and their complete effective configuration. For the System Prompt Filter, this includes the actual System Prompt text used during the original execution.

Moderari determines whether that recorded execution path can still be constructed from the currently available execution components. If a required recorded Filter or Adapter implementation/version no longer exists, Moderari rejects the Original Setup. It must not silently substitute a newer or different implementation. The user may then create a **New Setup**, which is resolved normally against the currently available execution environment. (To discuss - for new set-up - Repetere should link to Moderari to create the same)

The reconstruction therefore does not depend upon the continued existence of the original Filter Profile, saved System Prompt name, or other mutable configuration-time resources. It does depend upon the required recorded executable implementations still being available.

For a **New Setup replay**, Repetere does not modify or selectively override the recorded Session Execution State. A new session is configured through the normal Lumen configuration path by selecting a model and a Moderari Filter Profile, including explicit `None`. Praebere and Moderari then resolve the resulting execution environment normally, including Filter applicability, effective Filters and Adapter selection.

This is important because changing the replay model may itself change the applicable Filters or the Adapter required for execution. These dependent execution decisions are therefore resolved afresh rather than inherited from the original Session Execution State.

In both cases, the recorded client-boundary interaction remains unchanged.

> **Replay source material is the recorded client-boundary conversation. A replay either reconstructs the original recorded Session Execution State or creates a new Session Execution State through normal Lumen session configuration. Repetere does not mutate the recorded execution state.****

------------------------------------------------------------------------

# 21. Vestigare Remains Filter-Agnostic

Vestigare does not need to understand installed Filters, Filter-specific transformation schemas or intermediate Filter states.

Its raw-trace responsibility remains the exact client interaction/context, exact client-visible response, correlation/session identities and ordinary trace metadata.

Its generic execution-metadata facility may additionally retain the relevant materialised Moderari session setup, including resolved Filter and Adapter identity/version/configuration information.

Adding a new Filter must not require a Vestigare schema change specific to that Filter.

------------------------------------------------------------------------

# 22. Moderari Owns Filter Transformation Evidence

Moderari owns the Filter execution/application and transformation records that describe what its Filters actually did.

These records are distinct from both the Vestigare raw interaction trace and Vestigare generic execution metadata. Where transformation evidence is persisted, it must be correlatable with the raw trace and execution metadata, but it must not be inserted into the raw conversation.

The exact generic transformation-record schema remains an implementation design task.

# 23. Saved System Prompts

Saved System Prompts are user conveniences used to configure the System
Prompt Filter.

Prompt names must be unique.

Historical saved-prompt versions are not required for replay. The actual
resolved/effective System Prompt content used by the materialised Filter
configuration is execution truth and may be retained as generic execution
metadata. It is not substituted into the Vestigare raw conversation trace.

A saved prompt may later be edited or deleted without changing the
meaning of an existing trace.

------------------------------------------------------------------------

# 24. Separate Extension Specifications

The detailed extension contracts are intentionally separated from this
core redesign:

``` text
Lumen Moderari Adapter Architecture
    Canonical Interaction Model
    protocol/representation translation
    client/provider representation authority
    Adapter discovery/plugin contract
    Qwen Adapter initial scope

Lumen Moderari Filter Architecture
    Filter discovery/plugin contract
    System Prompt Filter
    Cognition Filter
    Filter Profiles
    capability requirements
    interaction transformation evidence
```

This keeps the Moderari redesign focused on service responsibilities and
session lifecycle while allowing the two extension systems to evolve
independently.

------------------------------------------------------------------------

# 28. Emerging Service Responsibilities

The resulting responsibility boundaries are becoming considerably
clearer.

## Praebere

**Question: What am I running?**

Responsibilities:

``` text
Provider
Model
Model characteristics
Capabilities
Availability
Execution target information
```

------------------------------------------------------------------------

## Moderari

**Question: How is the interaction transformed?**

Responsibilities:

``` text
Adapter framework
Client representation recognition
Adapter registry and resolution
Filter framework
Moderari Filter Profile catalogue and resolution
Client-support requirements
Filter discovery
Filter configuration
Filter validation
Filter execution
```

------------------------------------------------------------------------

## Rogare

**Question: How does the user select and configure it?**

Responsibilities:

``` text
Discover providers/models from Praebere
Discover Moderari Filter Profiles and filters from Moderari
Construct dynamic UI
Configure the existing Pontis session
Configure/create the session atomically
```

Rogare should not need model-specific or filter-specific UI code.

------------------------------------------------------------------------

## Vestigare

**Question: What was the true conversation?**

Responsibilities:

``` text
Exact client interaction/context presented to Lumen
Exact response returned to the client in the client interaction representation
Session/exchange correlation
Raw conversation trace
```

------------------------------------------------------------------------

## Repetere

**Question: Can I replay the raw interaction under a chosen execution
condition?**

Responsibilities:

``` text
Use the Vestigare raw trace as replay source material
Establish a requested Moderari session environment where required
Keep Filter-transformed context out of the raw replay source
Record replay as a new execution
```

------------------------------------------------------------------------

## Aestimare

**Question: What difference did it make?**

Responsibilities:

``` text
Assess results
Compare executions
Compare models
Compare filter configurations
Evaluate experimental outcomes
```

------------------------------------------------------------------------

# 29. Extension Requirements

Detailed Adapter and Filter requirements now live in their respective
architecture documents.

At the Moderari Core level, the required extension properties are:

``` text
Adapters
    discoverable
    independently installable
    self-describing
    automatically resolved
    canonical-representation based
    traceable generically

Filters
    discoverable
    independently installable
    self-describing
    session-scoped
    explicitly selected through Filter Profiles
    capability-gated
    traceable generically
```

Moderari Core must not require extension-specific branches for newly
installed compliant Adapters or Filters.

------------------------------------------------------------------------

# 30. Current Working Definition of Moderari

> **Moderari is Lumen's controlled interaction-transformation framework.**

Within that definition:

> **Pontis transports the client interaction and session identity. Moderari's Adapter Framework determines the client interaction representation from the request boundary.**

> **Praebere tells Moderari what provider/model is executing, the provider interaction representation, and the authoritative model characteristics.**

> **When Adapter processing is Enabled and canonical translation is required, base representation Adapters map published external representation semantics to/from Moderari's internal Canonical Interaction Model; provider/model-specific specialisations supply only known deviations from that base contract.**

> **The CIM belongs to the Adapter Framework. It is an internal semantic mapping model, not a new Lumen wire protocol and not the universal internal representation of Moderari.**

> **Filters perform explicit, user-selected transformations without constructing or consuming the CIM. The Filter Framework materialises and persists each Final Filter Output in the recognised client interaction representation, using Adapter Framework loopback where required.**

> **For the current redesign, Moderari retains the session conversation and model-facing context required by the existing interaction and Filter machinery. Client-owned full context is a later requirement.**

> **Moderari supplies no implicit default System Prompt. Any Lumen/user System Prompt is selected through a Filter Profile containing the System Prompt Filter.**

> **Zero Filters is a valid first-class configuration and represents Vanilla Moderari.**

> **The current supplied Filters are System Prompt and Cognition. The supplied base representation Adapters are OpenAI and Anthropic, with Qwen supplied as a specialised Adapter. The final Cognition behaviour is deferred until the existing implementation is inspected**

> **Adapters and Filters are independently installable, discoverable executable extensions. Filter Profiles are user-owned configuration convenience, not executable extensions.**

> **Vestigare retains the raw interaction trace and generic execution metadata describing the materialised execution environment. Filter execution artefacts are not part of the raw trace. Moderari separately persists Filter execution evidence, correlated by Moderari-owned Filter execution identity, describing what its Filters did.**

# 31. Architectural Acceptance Criteria

The Moderari Core redesign is successful when:

> **A session can be created by Pontis, bound to a provider/model by Praebere, configured with an explicit Filter Profile decision, configured before execution and finalised by Moderari on first Ask, optionally translated through an enabled Adapter/CIM path where compatibility requires it, transformed by the resolved Filter set over Moderari-managed textual/contextual interaction material, and executed without Moderari containing client-, provider-, model-, Adapter- or Filter-specific policy in its core.**

The extension-specific acceptance criteria are defined in the separate Adapter, Filter and CIM architecture documents.

# Additional Execution Resolution Requirements

## MCR-00A --- Client Representation Recognition

Moderari's Adapter Framework must determine the client interaction
representation from the ordinary request boundary using installed
representation definitions. Pontis must not be required to classify the AI
interaction representation.

If the representation cannot be positively distinguished, Moderari must retain
Unknown rather than guess.

## MCR-00B --- Representation Recognition Extension Boundary

Recognition of a newly supported external interaction representation should be
provided through the Moderari Adapter extension contract and must not require
representation-specific changes to Pontis or Moderari Core.


## MCR-00C --- Filter Representation Precondition

Filter Profile model applicability may be evaluated and a Profile may be selected during session configuration before the client interaction representation is known. On the first Ask, a selected Filter Profile may execute only if Moderari positively recognises the client representation and establishes the compatible representation path required for Filter execution. An Unknown client representation with No Filter Profile may pass through; an Unknown client representation with selected Filters must be rejected.

## MCR-00D --- Filter Execution Identity and Persistence

Moderari's Filter Framework must create and own a unique Filter execution identity for each Filter invocation. Moderari must persist the Final Filter Output after any required loopback processing in the recognised client interaction representation. Where a Filter performs private model inference, its private Ask and returned Response must also be persisted as execution artefacts associated with that Filter execution identity.

## MCR-01 --- Session Configuration and First-Ask Finalisation

During session configuration, Moderari must receive and retain the authoritative
reserved-session provider/model execution information and Effective Model
Definition supplied by Praebere, and resolve the selected Filter Profile to
its effective Filters/configuration.

The first Ask must not reacquire model knowledge already supplied during
configuration. It establishes only request-boundary facts that could not have
been known earlier, principally the client interaction representation and
resulting effective Adapter path. On successful resolution, Moderari finalises
the immutable Session Execution State used for subsequent Asks.

## MCR-02 --- No Provider Readiness Duplication

Moderari must not repeat provider health validation, model discovery,
model-loading readiness checks or model-knowledge resolution already
owned by Praebere.

------------------------------------------------------------------------

## MCR-03 --- Immutable Session Execution State

Moderari must finalise its session-scoped execution state on the first
Ask from the configuration already established before execution, and reuse that state for all subsequent Asks until the session ends.
The materialized provider/model characteristics, selected Filter
Profile, resolved Filters/configuration and resolved Adapter implementation/state
must not silently change during that execution session.

## MCR-03A --- Pre-Execution Filter Profile Association

When a Filter Profile decision is made for an existing Pontis session,
Rogare must send the selection and `session_id` to Moderari through
Nuntius using `\obt`. The selection may be a named Filter Profile or
explicit `None` for No Filter Profile. Moderari must retain the
association as pre-execution session configuration until the first Ask
finalises the Moderari Session Execution State or until the session ends.

## MCR-03B --- Configuration Lock After First Ask

After the first Ask finalises the Moderari Session Execution State, Moderari
must reject attempts to change the Filter Profile decision for that
executing session. A different execution configuration requires a new
session.

## MCR-03C --- Session-End Cleanup

When Pontis announces session end, Moderari must remove all live
session-scoped configuration and execution state for that `session_id`,
including an unused pre-execution Filter Profile association.

## MCR-04 --- Unconditional Vestigare Evidence

Moderari must emit defined execution evidence to Vestigare through
Nuntius using `\obt` regardless of whether Vestigare is recording.

## MCR-05 --- Recording-State Independence

Moderari must not query, infer, cache or depend upon Vestigare recording
state. Vestigare alone decides whether received evidence is persisted.

## MCR-06 --- Evidence Transport Separation

The Moderari→Nuntius→Vestigare evidence path must remain separate from
the Moderari→Pontis operational heartbeat/progress backchannel.
