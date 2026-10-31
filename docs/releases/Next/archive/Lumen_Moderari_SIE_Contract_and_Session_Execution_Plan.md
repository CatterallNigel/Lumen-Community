# Lumen Moderari ↔ SIE Contract and Session Execution Plan

**Status:** Current architecture / implementation contract  
**Date:** 02026-10-10  
**Scope:** Moderari Standalone Interaction Engine (SIE), session establishment, execution planning, Adapter and Filter resolution, immutable Session Execution Plans, Vestigare registration, and replay materialisation.

---

## 1. Purpose

This document defines the contract between **Moderari Core** and the **Standalone Interaction Engine (SIE)** for establishing and executing a Lumen interaction session.

It also defines the ownership, construction, immutability, execution, evidence and replay semantics of the **Session Execution Plan (SEP)**.

The governing responsibility boundary is:

> **Moderari owns the Session Execution Plan as part of the Moderari session. SIE creates and executes an `ADAPTERS_ENABLED` Plan. Moderari creates the minimal `ADAPTERS_DISABLED` Plan because SIE is not involved in that execution path.**

Accordingly:

- Moderari owns the overall session lifecycle, user choices, Filter Profiles, provider transport, Vestigare registration and session `READY` state.
- Praebere owns provider/model facts and expresses them through the Effective Model Definition (EMD).
- SIE is the flow-control gateway when Adapter processing is enabled.
- SIE discovers and resolves executable components, determines candidate Filter availability, constructs and freezes the enabled SEP, materialises its runtime Execution Instance, and executes the Plan.
- Vestigare must receive the immutable SEP before Moderari permits normal interaction execution.
- Repetere reuses the recorded immutable SEP; it does not recreate the decisions that produced it.

---

## 2. Architectural Invariants

### 2.1 Moderari owns the SEP; SIE creates and executes enabled Plans

> **Moderari owns the Session Execution Plan. SIE is authoritative for constructing, validating, finalising and executing an `ADAPTERS_ENABLED` Plan. Moderari MUST NOT alter a Plan returned by SIE.**

For `ADAPTERS_DISABLED`, there is no Moderari ↔ SIE communication. Moderari creates the minimal disabled SEP and registers it with Vestigare.

### 2.2 Every executable Moderari session has a SEP

A SEP has exactly one of two execution modes:

```text
Session Execution Plan
        │
        ├── ADAPTERS_DISABLED
        │      direct Moderari/provider path
        │      no SIE execution
        │      no CIM
        │      no Filters
        │
        └── ADAPTERS_ENABLED
               complete immutable SIE execution definition
```

> **`ADAPTERS_DISABLED` is a valid and complete execution plan, not the absence of a Plan.**

This is required so Vestigare and Repetere can distinguish deliberate direct execution from an enabled Adapter/CIM/Filter execution path.

### 2.3 A SEP exists only in finalised form

SIE may maintain mutable configuration or plan-construction state while an enabled session is being prepared. That mutable state is **not** a Session Execution Plan.

> **A `SessionExecutionPlan` exists only after finalisation and is immutable for its entire lifetime.**

The same immutable object may be used by:

- the active Moderari session;
- Vestigare as execution provenance; and
- Repetere as the historical execution definition for replay.

### 2.4 SIE is flow control

> **SIE always takes the output of the current execution action and supplies it as the input to the next action in the immutable SEP execution flow until the terminal Adapter operation produces the external representation returned to Moderari.**

SIE does not perform protocol translation itself. Adapters translate representations. Filters consume and emit CIM.

### 2.5 Adapters are boundary identities, not permanent directions

The SEP identifies a **client Adapter** and a **provider Adapter**. `IN` and `OUT` describe the operation invoked at a particular point in a flow; they are not Adapter identities.

```text
                     TO MODEL             FROM MODEL

Client Adapter A       IN                    OUT
Provider Adapter B     OUT                   IN
```

Normal to-model flow:

```text
Protocol A
   ↓
Client Adapter A.IN
   ↓
CIM
   ↓
Filter A
   ↓
Filter B
   ↓
Provider Adapter B.OUT
   ↓
Protocol B
```

Normal from-model flow:

```text
Protocol B
   ↓
Provider Adapter B.IN
   ↓
CIM
   ↓
Filter B
   ↓
Filter A
   ↓
Client Adapter A.OUT
   ↓
Protocol A
```

### 2.6 Filters operate only on CIM

> **Filters consume CIM and emit CIM.**

There is no Filter loopback architecture and no protocol-specific Filter path.

A Filter-enabled session therefore requires a valid CIM path between the client and provider boundaries.

### 2.7 Filter order is execution semantics

> **Filter order MUST be supplied by Moderari and MUST be preserved exactly by SIE.**

The order originates in the Moderari-owned Filter Profile selected by the user. SIE validates the resolved Filters and configuration but MUST NOT reorder or substitute them.

For a Plan containing:

```text
filters = [A, B, C]
```

the primary traversal is:

```text
TO MODEL:   A → B → C
FROM MODEL: C → B → A
```

The ordered Filter chain is therefore part of the immutable execution semantics and provenance of the SEP.

### 2.8 Private Filter execution is correlated by `filter_execution_id`

A Filter that performs a private Ask creates its own opaque `filter_execution_id` in CIM. SIE does not maintain an ID-to-Filter ownership table.

The private Ask leaves through the provider Adapter:

```text
Originating Filter
   ↓
CIM + filter_execution_id
   ↓
SIE
   ↓
Provider Adapter.OUT
   ↓
Moderari → Provider
```

The provider result returns as:

```text
Provider
   ↓
Moderari
   ↓
Provider Adapter.IN
   ↓
CIM + filter_execution_id
   ↓
reverse Filter traversal
```

Each Filter determines whether it owns the identifier. A Filter receiving a private execution identifier it does not own passes the CIM back to SIE unchanged, allowing reverse traversal to continue.

> **A CIM carrying a `filter_execution_id` MUST be claimed by exactly one Filter in the session's immutable Filter chain. If reverse traversal completes with the identifier unclaimed, SIE MUST treat the execution as failed rather than allowing the private response to escape through the client Adapter.**

### 2.9 Client interface is immutable within a session

Moderari infers the client interface from the first Ask and supplies it using SIE's canonical interface enum.

> **The client interface is write-once for an enabled session and cannot change protocol/interface during that session.**

### 2.10 `ADAPTERS_ENABLED` means SIE has actual work to perform

The availability of suitable Adapters does not by itself require an enabled SIE execution path.

Once the client interface is known, SIE may determine that the client and provider interfaces are already the same, the provider uses only its Base Adapter, no SpAd is required, and the user selected no Filters. In that case canonicalising to CIM and materialising back to the same interface would add no execution function.

> **`ADAPTERS_ENABLED` means that SIE has an actual representation-correction or CIM-processing responsibility for the session.**

When no such responsibility exists, SIE returns `ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED`, discards its construction state, and Moderari creates the immutable `ADAPTERS_DISABLED` SEP for direct pass-through.

### 2.11 Vestigare registration is mandatory before Moderari `READY`

> **Every executable Moderari session MUST have its immutable SEP successfully registered with Vestigare before Moderari transitions the session to `READY`.**

Vestigare registration is not deferred metadata.

---

## 3. Responsibility Boundaries

### 3.1 Moderari Core owns

Moderari owns:

- the overall Lumen session and its lifecycle;
- the Session Execution Plan as part of the session execution definition;
- the user's Adapter Enabled/Disabled choice;
- communication with Praebere;
- receipt and transport of the EMD;
- client-interface inference from the first Ask;
- Filter Profile storage and management;
- determination of which Filter Profiles can be offered from SIE's candidate Filter set;
- user selection of a Filter Profile or explicit `None`;
- resolution of the selected Profile into an ordered Filter list and configuration;
- provider transport/execution;
- mandatory SEP registration with Vestigare;
- the overall Moderari session `READY` state;
- routing requests and responses between client, SIE and provider.

### 3.2 SIE owns when Adapter processing is enabled

SIE owns:

- transient SIE session/configuration state keyed by the Moderari `session_id`;
- component discovery/catalogue;
- Base Adapter and Specialised Adapter resolution;
- Filter discovery and applicability evaluation;
- provider-side Adapter resolution sufficient to determine candidate Filter availability;
- validation of Moderari's resolved ordered Filter selection;
- final client-side Adapter resolution when the first Ask establishes the client interface;
- complete execution-path validation;
- construction, validation and finalisation of the enabled SEP;
- runtime Execution Instance materialisation;
- execution of the immutable Plan;
- exact validation and materialisation of a recorded enabled SEP supplied for replay.

SIE does **not** own the overall Moderari session `READY` state.

### 3.3 Praebere owns

Praebere owns provider/model knowledge. The EMD is the authoritative carrier of those facts and should include at least:

- provider identity;
- model identity;
- provider/model interaction interface using the canonical interface identity required by Moderari/SIE;
- model characteristics required for Filter applicability and execution decisions.

Moderari SHOULD transport the EMD to SIE rather than decomposing and restating Praebere-owned facts as duplicate SIE parameters.

### 3.4 Vestigare owns

Vestigare owns the evidence record describing the immutable execution environment and subsequent interaction evidence.

The registered SEP is the authoritative declaration of how the session was configured to execute.

### 3.5 Repetere owns

Repetere owns replay orchestration, not reconstruction of historical execution decisions.

> **Repetere supplies the entire recorded SEP to Moderari. It does not resolve current Adapters, current Filters or a current equivalent execution configuration.**

---

## 4. The Two Session Establishment Paths

### 4.1 `ADAPTERS_DISABLED`

When Adapter processing is disabled, SIE is not involved at all.

```text
Moderari
   ↓
create immutable SEP:
    execution_mode = ADAPTERS_DISABLED
   ↓
Moderari → Vestigare: register SEP
   ↓
Vestigare accepts
   ↓
SIE materialises Execution Instance
   ↓
Moderari session → READY
   ↓
Client ↔ Moderari ↔ Provider
```

There is:

- no SIE session;
- no EMD exchange with SIE;
- no Adapter resolution by SIE;
- no CIM path;
- no Filter availability from SIE;
- no Execution Instance in SIE.

### 4.2 SIE-capable construction and final execution-mode decision

When a provider Adapter exists, Moderari establishes temporary SIE construction state. This does not yet imply that the final SEP will be `ADAPTERS_ENABLED`.

```text
Moderari creates overall session
   ↓
Moderari → SIE: create SIE session
   ↓
Moderari → SIE: configure_provider(session_id, emd)
   ↓
SIE resolves provider Adapter
   │
   ├── unavailable
   │      ↓
   │   no provider-side CIM boundary / no Filters
   │      ↓
   │   SIE construction ends
   │      ↓
   │   Moderari creates ADAPTERS_DISABLED SEP
   │      ↓
   │   direct provider pass-through
   │
   └── available
          ↓
      SIE returns available Filters
          ↓
      Moderari offers matching Filter Profiles + Filters:None
          ↓
      Moderari → SIE: ordered selected Filters + configuration
          ↓
      First Ask arrives and is held by Moderari
          ↓
      Moderari identifies canonical client interface
          ↓
      Moderari → SIE: set_client_interface(...)
          ↓
      SIE evaluates whether an enabled SIE path is required
          │
          ├── ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
          │      ↓
          │   ERROR → end session
          │
          ├── ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
          │      ↓
          │   discard SIE construction state
          │      ↓
          │   Moderari creates ADAPTERS_DISABLED SEP
          │      ↓
          │   Vestigare → READY → direct pass-through
          │
          └── ADAPTER_ENABLED
                 ↓
             SIE validates complete execution definition
                 ↓
             SIE finalises immutable ADAPTERS_ENABLED SEP
                 ↓
             SIE → Moderari: immutable SEP
                 ↓
             Moderari → Vestigare: register SEP
                 │
                 ├── failure → terminate session; discard SIE state
                 └── accepted
                        ↓
                    materialise_execution(session_id)
                        ↓
                    Execution Instance
                        ↓
                    Moderari session → READY
                        ↓
                    held first Ask may execute
```

`ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED` applies when the client and provider interfaces are the same, the provider requires no SpAd, and the user selected no Filters. SIE therefore has no representation-correction or CIM-processing responsibility.

A provider SpAd or any selected Filter gives SIE work to perform even when the client and provider interfaces are identical.

No Execution Instance is materialised for either `ADAPTERS_DISABLED` outcome.

---

## 5. Enabled SIE Session Construction State

An enabled SIE session needs transient construction state before a SEP exists.

Conceptually:

```text
SIESessionConstruction
│
├── session_id
├── state
│    CREATED
│    MODEL_CONFIGURED
│    FILTERS_CONFIGURED
│    PLAN_PREPARED
│    CLOSED
│
├── emd
├── provider_adapter_candidate
├── candidate_filters
├── selected_filters[]
├── client_interface
├── client_adapter_candidate
├── finalised_plan
└── execution_instance
```

The exact implementation names may differ, but the semantic distinctions are required.

`FILTERS_CONFIGURED` is distinct from “no Filter decision yet.” An explicit Moderari `None` selection is represented by an ordered empty Filter list.

`PLAN_PREPARED` means SIE has finalised an immutable enabled SEP and successfully materialised the corresponding Execution Instance. It does **not** mean the Moderari session is `READY`.

---

## 6. Provider/Model Configuration Through the EMD

For enabled processing, Moderari supplies the Praebere-owned EMD to SIE.

Conceptual operation:

```text
configure_provider(
    session_id,
    emd
)
```

Provider identity, model identity and provider/model interface SHOULD be read from the EMD rather than duplicated as independent parameters.

The provider/model configuration is write-once for the SIE session. Changing provider/model execution identity requires a new execution-plan construction rather than mutation of a final SEP.

At this stage SIE does not yet know the client interface and MUST NOT claim that the complete client↔provider Adapter path has been established.

---

## 7. Provider Adapter Resolution and Candidate Filter Availability

After receiving the EMD, SIE resolves the effective provider boundary Adapter using the canonical provider interface plus provider/model identity contained in the EMD.

This may resolve a Base Adapter or a matching Specialised Adapter.

Conceptually:

```text
EMD
 ├── provider identity
 ├── model identity
 └── provider interface
          ↓
    Adapter resolution
          ↓
 effective provider Adapter
```

This resolution is sufficient to determine whether selected Filters could eventually emit CIM that can be materialised toward the provider/model.

If no provider Adapter can be resolved, candidate Filter availability is empty.

If a provider Adapter resolves, SIE evaluates discovered Filters against:

- the active CIM version;
- Filter-declared EMD `requires`; and
- the existence of the provider-side CIM boundary.

Conceptual result:

```text
candidate_filters(session_id)
    → [FilterIdentityVersion, ...]
```

This is **candidate** availability because the client-side Adapter cannot be validated until the first Ask establishes the client interface.

---

## 8. Filter Profiles and Ordered Filter Selection

Moderari owns Filter Profiles. SIE does not receive or execute Profile names.

Moderari compares its Profiles with the candidate Filter set and determines which Profiles may be offered to the user. `None` is always an explicit Filter selection meaning an ordered empty Filter chain.

After selection, Moderari supplies the resolved ordered Filter definitions and configuration:

```text
configure_filters(
    session_id,
    [
        Filter A identity/version + configuration,
        Filter B identity/version + configuration,
        Filter C identity/version + configuration
    ]
)
```

For `None`:

```text
configure_filters(session_id, [])
```

SIE MUST validate that every supplied Filter:

- exists at the required identity/version;
- is compatible with the active CIM version;
- was in the candidate Filter set;
- satisfies its EMD requirements; and
- accepts the supplied configuration.

SIE MUST preserve the supplied order exactly.

---

## 9. First Ask and Client Interface Establishment

The first Ask establishes the client interface for an SIE-capable construction.

Moderari recognises the client interface and passes the canonical SIE interface enum/identity while holding the Ask.

Conceptually:

```text
set_client_interface(
    session_id,
    Interface.<canonical>
) -> AdapterStatusResult
```

with:

```text
AdapterStatus
    ADAPTER_ENABLED
    ADAPTER_DISABLED

AdapterStatusReason
    ADAPTER_NOT_AVAILABLE
    ADAPTER_NOT_REQUIRED
```

The client interface is write-once.

SIE resolves the Base client Adapter and evaluates the complete construction state: client interface, provider interface, provider Base Adapter/SpAd, and selected Filters.

The outcomes are:

```text
ADAPTER_ENABLED
    → finalise_execution_plan()
    → ADAPTERS_ENABLED SEP

ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
    → ERROR
    → Moderari ends session

ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
    → discard SIE construction
    → Moderari creates ADAPTERS_DISABLED SEP
    → direct pass-through
```

`ADAPTER_NOT_AVAILABLE` means an enabled SIE path is required but the required client Base Adapter cannot be resolved.

`ADAPTER_NOT_REQUIRED` means SIE has determined that no enabled path is necessary because the client and provider interfaces are the same, the provider requires no SpAd, and `selected_filters[]` is empty.

A SpAd or any selected Filter prevents the `ADAPTER_NOT_REQUIRED` outcome because SIE then has representation-correction or CIM-processing work to perform.

---

## 10. Complete Execution-Path Validation

For a non-empty Filter chain, the enabled Plan requires both boundary Adapters because Filters operate on CIM.

```text
Client Protocol A
      ↓
Client Adapter A.IN
      ↓
     CIM
      ↓
ordered Filters
      ↓
     CIM
      ↓
Provider Adapter B.OUT
      ↓
Provider Protocol B
```

The reverse boundary must also be valid:

```text
Provider Protocol B
      ↓
Provider Adapter B.IN
      ↓
     CIM
      ↓
reverse ordered Filters
      ↓
     CIM
      ↓
Client Adapter A.OUT
      ↓
Client Protocol A
```

If the selected enabled execution definition cannot be materialised without inventing or substituting components, finalisation fails and no SEP is produced.

There is no Filter loopback fallback.

---

## 11. Session Execution Plan Forms

### 11.1 Disabled Plan

A minimal disabled Plan is sufficient to reproduce the deliberate direct execution mode.

```yaml
session_id: "..."
execution_mode: ADAPTERS_DISABLED
```

Additional general provenance fields may be added, but Adapter/CIM/Filter execution fields are not required because they were not part of the execution path.

### 11.2 Enabled Plan

A conceptual enabled Plan is:

```yaml
session_id: "..."
execution_mode: ADAPTERS_ENABLED

cim_version: "1.0"

client:
  interface: openai-chat-completions

client_adapter:
  identity: openai-chat-completions-base
  version: "..."

emd:
  # authoritative Praebere provider/model execution definition
  ...

provider_adapter:
  identity: qwen-tool-spad
  version: "..."
  base_adapter: openai-chat-completions-base

filters:
  - identity: system-prompt
    version: "..."
    configuration:
      prompt: "..."

  - identity: cognition
    version: "..."
    configuration:
      ...
```

The `filters` array is ordered and normative.

The final implementation schema may add provenance or validation metadata, but it MUST preserve the ownership, exact-version and immutability rules in this document.

---

## 12. SEP Finalisation

For a normal enabled session, SIE performs:

```text
configuration inputs
      ↓
component resolution
      ↓
validation
      ↓
FINALISE
      ↓
immutable SessionExecutionPlan
      ↓
materialise Execution Instance
```

Finalisation is one-way.

> **Once finalised, no field in a SEP may be modified by Moderari, SIE, Vestigare or Repetere.**

If the intended execution environment changes, it requires a new Plan/session execution definition; it does not mutate the existing SEP.

---

## 13. Runtime Execution Instance

The Execution Instance is the transient runtime materialisation of an immutable enabled SEP.

Conceptually:

```text
ExecutionInstance
│
├── client_adapter_instance
├── filter_instances[]       # exact SEP order
└── provider_adapter_instance
```

The Execution Instance may contain runtime state required to execute Filters and Adapter operations, but it MUST NOT replace the immutable identities, versions and configuration recorded in the SEP.

> **Materialisation MUST NOT modify the SEP.**

---

## 14. Primary SIE Flow Execution

SIE executes the Plan as an ordered flow.

### 14.1 Client → model

```text
input = client external representation

Client Adapter.IN(input)
        ↓
       CIM
        ↓
Filter A
        ↓
Filter B
        ↓
Filter C
        ↓
Provider Adapter.OUT
        ↓
provider external representation
        ↓
return to Moderari
```

### 14.2 Model → client

```text
input = provider external representation

Provider Adapter.IN(input)
        ↓
       CIM
        ↓
Filter C
        ↓
Filter B
        ↓
Filter A
        ↓
Client Adapter.OUT
        ↓
client external representation
        ↓
return to Moderari
```

SIE does not need protocol-specific branching in its flow engine. The Plan and materialised components determine each action.

---

## 15. Private Filter Ask/Response Flow

A Filter may perform a private Ask using the same provider boundary.

The originating Filter creates an opaque `filter_execution_id` and emits a CIM private Ask.

```text
Filter B
   ↓
CIM + filter_execution_id = XYZ
   ↓
SIE
   ↓
Provider Adapter.OUT
   ↓
Moderari → Provider
```

The provider response returns through the provider Adapter:

```text
Provider
   ↓
Moderari
   ↓
SIE
   ↓
Provider Adapter.IN
   ↓
CIM + XYZ
   ↓
reverse Filter traversal
```

Example for `[A, B, C]` where B owns `XYZ`:

```text
C → does not own XYZ → pass unchanged to SIE
B → owns XYZ          → process private result
```

SIE does not need to know in advance which Filter created `XYZ`.

If reverse traversal reaches the beginning of the Filter chain with `filter_execution_id` still unclaimed, SIE MUST fail the private execution and MUST NOT materialise that CIM through the client Adapter.

---

## 16. Vestigare Registration and Moderari `READY`

### 16.1 Disabled

```text
Moderari creates immutable ADAPTERS_DISABLED SEP
        ↓
Moderari → Vestigare
        ↓
Vestigare accepts
        ↓
Moderari session → READY
```

### 16.2 Enabled

```text
SIE finalises immutable ADAPTERS_ENABLED SEP
        ↓
SIE materialises Execution Instance
        ↓
SIE → Moderari: frozen SEP
        ↓
Moderari → Vestigare
        ↓
Vestigare accepts
        ↓
Moderari session → READY
```

> **Moderari owns `READY`. SIE does not transition the overall session to `READY`.**

If Vestigare registration fails, Moderari MUST NOT mark the session `READY` and MUST NOT release the first Ask for provider execution under that unregistered Plan.

SIE does not require a Vestigare acknowledgement; Moderari enforces the gate by withholding normal execution traffic until registration succeeds.

---

## 17. Normal Runtime Contract After `READY`

For an enabled session, subsequent execution calls should be deliberately small:

```text
process_request(session_id, request)
```

and:

```text
process_response(session_id, response)
```

SIE obtains the immutable SEP and its Execution Instance from the active SIE session and executes the appropriate traversal.

Moderari performs provider transport/execution.

No Adapter, Filter or version resolution occurs per Ask.

For a disabled session, Moderari does not call SIE; traffic follows the direct Moderari/provider path recorded by the disabled SEP.

---

## 18. Replay Contract

Repetere obtains the historical immutable SEP from evidence and sends the **entire Plan** to Moderari.

### 18.1 Recorded `ADAPTERS_DISABLED`

```text
Repetere → Moderari: recorded SEP
        ↓
execution_mode = ADAPTERS_DISABLED
        ↓
recreate direct execution mode
        ↓
SIE not involved
```

### 18.2 Recorded `ADAPTERS_ENABLED`

```text
Repetere → Moderari: recorded SEP
        ↓
Moderari → SIE: entire recorded SEP
        ↓
SIE exact validation
        ↓
materialise Execution Instance
        ↓
execute replay
```

A recorded Plan bypasses **resolution**, but it does not bypass **validation**.

SIE MUST validate exact availability of:

- the recorded CIM version;
- the recorded client Adapter identity and version;
- every recorded Filter identity and version;
- the exact ordered Filter configuration;
- the recorded provider Adapter identity and version; and
- any other execution component made normative by the SEP schema.

If any exact required component is unavailable, SIE MUST reject the SEP as not faithfully executable.

SIE MUST NOT:

- substitute a newer Adapter version;
- fall back from a recorded SpAd to a Base Adapter;
- substitute a Filter version;
- reorder Filters;
- replace recorded Filter configuration with current Profile configuration;
- upgrade or downgrade the CIM version;
- re-resolve what it would choose today; or
- mutate the recorded SEP.

> **Repetere replays the Session Execution Plan; it does not recreate the decisions that produced it.**

---

## 19. Exact Contract Types — Architectural Shape

The following types describe the contract to be implemented. Names may be adjusted to fit the existing codebase, but their semantics should remain.

### 19.1 Execution mode

```text
ExecutionMode
    ADAPTERS_DISABLED
    ADAPTERS_ENABLED
```

`ADAPTERS_ENABLED` is the default assumption while an SIE-capable session is being configured. The execution mode becomes an immutable execution fact only when the SEP is finalised.

If no provider Adapter can be resolved, there is no provider-side CIM boundary and therefore no Filter can execute. SIE construction ends and Moderari immediately uses `ADAPTERS_DISABLED` for the session. The client interface is irrelevant to that SIE decision.

A second `ADAPTERS_DISABLED` outcome may be determined later, after the client interface is known. If the client and provider interfaces are the same, the provider requires no SpAd, and `selected_filters[]` is empty, SIE returns `ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED`. No enabled SEP is created; SIE discards its construction state and Moderari creates the disabled SEP.

### 19.2 Immutable final SEP

```text
SessionExecutionPlan
    session_id
    execution_mode

    # present only when ADAPTERS_ENABLED
    cim_version
    client_interface
    provider_interface
    provider
    model
    client_adapter
    provider_adapter
    ordered_filters[]
```

The schema MUST enforce the execution-mode discriminator:

- `ADAPTERS_DISABLED` is complete with `session_id` and `execution_mode`; SIE execution fields are absent because SIE did not participate.
- `ADAPTERS_ENABLED` requires a complete SIE execution definition. Both client and provider Adapter references MUST exist. `ordered_filters[]` may be empty.
- An `ADAPTERS_ENABLED` SEP MUST NOT contain `None` in place of either boundary Adapter.
- The full EMD is not part of the SEP. The EMD is construction-time input used by SIE to resolve and validate the execution environment. The SEP records the resulting execution facts required for provenance, execution and replay.

`provider` and `model` record the original execution target as provenance. `provider_interface` records the provider protocol/interface against which the provider Adapter was materialised.

For replay, the selected replay provider/model need not be identical to the original provider/model, but the replay target MUST expose the `provider_interface` recorded in the SEP. SIE MUST use the recorded provider Adapter rather than resolving a replacement.

If the recorded provider Adapter is a Specialised Adapter (SpAd), the replay provider/model MUST also satisfy that SpAd's declared provider/model or model-family applicability. A replay MAY therefore move, for example, the same applicable model family between providers when the SpAd permits that provider scope and the provider interface remains the same. It MUST fail when the selected replay model/provider does not satisfy the recorded SpAd specification.

> **Replay validates the recorded execution path against the selected replay target; it does not re-resolve a different execution path.**

### 19.3 Adapter reference

```text
AdapterReference
    identity
    version
    cim_version

    # present for a Specialised Adapter
    base_adapter?
        identity
```

A Specialised Adapter is a **provider-boundary Adapter**. The client boundary resolves only the Base Adapter for the canonical client interface. A SpAd may specialise provider-, model-, model-family-, or provider/model-specific representation behaviour and may override provider-boundary `IN`, `OUT`, or both.

A Specialised Adapter depends on the identity and CIM compatibility of its Base Adapter, not on an exact Base Adapter implementation version.

A Base Adapter implementation may therefore receive corrections, extensions, protocol-coverage improvements or internal changes without invalidating every SpAd that extends it, provided it continues to target the same CIM version and preserves the applicable Base Adapter contract.

A CIM-version change is a deliberate compatibility boundary and is handled separately as part of a Lumen/CIM upgrade decision.

The SpAd's own identity and version remain exact execution provenance and MUST be available when materialising a recorded SEP.

### 19.4 Filter execution reference

```text
FilterExecutionReference
    identity
    version
    configuration
```

Array position is execution order.

`configuration` means the runtime initialisation/configuration supplied to that Filter for this execution. For example, a System Prompt Filter records the configured initial prompt. Model characteristics or Filter `requires` declarations are not duplicated into the SEP; they belong to the Filter definition and are evaluated from the relevant model definition when required.

### 19.5 Enabled SIE construction state

```text
SIESessionConstruction
    session_id

    provider
    model
    provider_interface
    provider_adapter

    selected_filters[]

    client_interface?
    client_adapter?

    finalised_plan?
```

The EMD is construction input and need not be retained wholesale after SIE has extracted the provider/model/interface facts and model characteristics required for Adapter resolution and Filter applicability.

The Filters that conform to the selected model's attributes are a **discovery result**, not SIE session state. SIE evaluates Filter applicability and returns the resulting available Filter identities to Moderari. SIE has no need to retain that availability result: Moderari may find no matching Filter Profile, and the user may explicitly select `Filters: None`.

Moderari owns Filter Profiles and user selection. After the user selects a Profile or `Filters: None`, Moderari resolves that choice to the exact ordered Filter configuration and supplies `selected_filters[]` to SIE. An empty array is the explicit no-Filter selection.

Provider Adapter resolution is the prerequisite for continued SIE participation:

```text
provider Adapter unavailable
        ↓
no provider-side CIM boundary
        ↓
no Filters can be offered or executed
        ↓
SIE construction ends
        ↓
Moderari → ADAPTERS_DISABLED
```

No client-interface decision is required for that SIE outcome.

If provider Adapter resolution succeeds, SIE retains the provider Adapter and subsequently the ordered selected Filters. When the first Ask reaches Moderari, Moderari identifies the canonical client interface and supplies it to SIE. SIE then resolves the client Adapter and, if the complete path is valid, creates the final immutable SEP.

After the client interface is known, SIE returns an `AdapterStatusResult`.

- `ADAPTER_ENABLED` means SIE has a complete path and actual SIE work to perform.
- `ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE` means an enabled path is required but the client Base Adapter is unavailable; Moderari returns a protocol-compatibility error and ends the session.
- `ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED` means no SIE work is required; SIE discards construction state and Moderari creates the `ADAPTERS_DISABLED` SEP for direct pass-through.

The `ADAPTER_NOT_AVAILABLE` outcome MUST NOT silently fall back to direct execution.

A failure before SEP finalisation produces no SEP. A finalised SEP exists before its Execution Instance and remains immutable while Vestigare registration is pending.

### 19.6 Execution Instance

```text
ExecutionInstance
    client_adapter_instance
    filter_instances[]
    provider_adapter_instance
```

An `ExecutionInstance` exists only for an `ADAPTERS_ENABLED` SEP that has been successfully accepted by Vestigare.

SEP finalisation and Execution Instance materialisation are separate operations:

```text
complete SIE configuration
        ↓
finalise immutable SEP
        ↓
Moderari registers SEP with Vestigare
        │
        ├── failure
        │      ↓
        │   Moderari terminates session
        │      ↓
        │   SIE destroys active SEP/session state
        │      ↓
        │   no Execution Instance
        │
        └── accepted
               ↓
        SIE materialises Execution Instance
```

Both Adapter instances are mandatory. `filter_instances[]` may be empty and MUST preserve the exact order recorded in the SEP.

For `ADAPTERS_DISABLED`, SIE does not execute the session and no `ExecutionInstance` exists.

The Execution Instance is the runtime flow materialisation of the immutable SEP:

```text
TO MODEL
client_adapter.IN
    ↓
filter_instances[0..n]
    ↓
provider_adapter.OUT

FROM MODEL
provider_adapter.IN
    ↓
filter_instances[n..0]
    ↓
client_adapter.OUT
```

Private Filter execution follows the same immutable Filter chain and the `filter_execution_id` ownership/reverse-traversal rules defined earlier in this contract.

---

## 20. Exact State Transitions

### 20.1 Moderari — disabled path

```text
SESSION_CREATED
      ↓
Moderari creates immutable SEP:
    execution_mode = ADAPTERS_DISABLED
      ↓
Moderari → Vestigare: register SEP
      ├── failure → terminate / establishment failure
      └── success
             ↓
           READY
             ↓
      direct Moderari execution
             ↓
           CLOSED
```

Moderari sends the `ADAPTERS_DISABLED` SEP to Vestigare. No SIE state exists for this session and there is no Moderari ↔ SIE communication on the disabled path.

### 20.2 SIE — enabled plan construction

```text
SIE SESSION CREATED
        ↓
receive provider/model EMD
        ↓
resolve provider Adapter
        │
        ├── unavailable
        │      ↓
        │   return no SIE path / no Filters
        │      ↓
        │   discard SIE construction state
        │      ↓
        │   Moderari → ADAPTERS_DISABLED
        │
        └── available
               ↓
        evaluate Filter applicability
        against model attributes
               ↓
        return available Filters to Moderari
               ↓
        [availability result need not be retained by SIE]
               ↓
        Moderari offers matching Filter Profiles + Filters:None
               ↓
        user selects Profile / None
               ↓
        Moderari resolves selection to ordered Filters
               ↓
        Moderari → SIE: selected_filters[]
               ↓
        SIE retains provider Adapter + selected Filters
               ↓
        wait for canonical client interface
```

At this point no SEP exists. The returned Filter availability set is discovery output only. It is not part of SIE's persistent construction state and does not constrain Moderari to select any Filter.

When the first Ask arrives, Moderari identifies the client interface and continues finalisation:

```text
Moderari → SIE: canonical client_interface
        ↓
SIE resolves client Base Adapter and evaluates whether SIE is required
        │
        ├── ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
        │      ↓
        │   protocol compatibility ERROR
        │      ↓
        │   end session / no SEP
        │
        ├── ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
        │      ↓
        │   discard SIE construction state
        │      ↓
        │   Moderari creates ADAPTERS_DISABLED SEP
        │      ↓
        │   direct pass-through after Vestigare registration
        │
        └── ADAPTER_ENABLED
               ↓
        validate complete execution definition
               ↓
        finalise immutable ADAPTERS_ENABLED SEP
               ↓
        return SEP to Moderari
               ↓
        wait for Vestigare registration result
```

SIE MUST NOT materialise the Execution Instance at SEP finalisation time.

### 20.3 Moderari — enabled path

```text
SESSION_CREATED
      ↓
provider/model selected
      ↓
SIE configuration and Filter discovery/selection
      ↓
first Ask arrives at Moderari
      ↓
Moderari identifies canonical client interface
      ↓
Moderari HOLDS first Ask
      ↓
Moderari → SIE: client_interface
      ↓
SIE returns AdapterStatusResult
      │
      ├── ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
      │      → ERROR → end session
      │
      ├── ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
      │      → discard SIE construction
      │      → Moderari creates/registers ADAPTERS_DISABLED SEP
      │      → READY → direct pass-through
      │
      └── ADAPTER_ENABLED
             ↓
SIE validates complete execution path
      ↓
SIE creates FINAL IMMUTABLE SEP
      ↓
SIE → Moderari: SEP
      ↓
Moderari → Vestigare: register SEP
      │
      ├── failure / reject
      │      ↓
      │   Moderari terminates session
      │      ↓
      │   Moderari tells SIE to terminate
      │      ↓
      │   SIE destroys active SEP/session state
      │      ↓
      │   no Execution Instance
      │      ↓
      │   CLOSED
      │
      └── accepted
             ↓
      Moderari → SIE: SEP accepted
             ↓
      SIE materialises Execution Instance
             │
             ├── failure → establishment failure / terminate session
             └── success
                    ↓
             Moderari session → READY
                    ↓
             release held first Ask to SIE
                    ↓
                 EXECUTION
                    ↓
                  CLOSED
```

The first Ask is used by Moderari to determine the client interface, but it MUST NOT execute through SIE or reach the provider until the immutable SEP has been accepted by Vestigare and the corresponding Execution Instance has been successfully materialised.

> **SEP finalisation precedes Vestigare registration. Execution Instance materialisation follows successful Vestigare registration.**

If Vestigare rejects or fails to register the SEP, Moderari terminates the session and instructs SIE to destroy the active SEP/session state. The SEP is never mutated; it simply never becomes the runtime basis of an Execution Instance.

### 20.4 Replay — enabled path

For replay, the client execution boundary is already defined by the recorded immutable SEP. The client interface and recorded client Adapter are therefore not re-discovered or re-resolved as part of replay establishment. Replay validation at this stage concerns the selected replay provider/model and its ability to execute the recorded provider-side plan.

```text
RECORDED IMMUTABLE SEP
        ↓
client execution boundary already defined by SEP
        ↓
selected replay provider/model
        ↓
provider interface matches recorded SEP?
        ├── NO → REPLAY FAIL
        └── YES
               ↓
recorded provider Adapter available?
        ├── NO → REPLAY FAIL
        └── YES
               ↓
recorded provider Adapter is SpAd?
        │
        ├── YES
        │      ↓
        │   replay provider/model satisfies
        │   recorded SpAd applicability?
        │      ├── NO → REPLAY FAIL
        │      └── YES
        │
        └── Base Adapter
               ↓
validate remaining recorded execution components
               ↓
materialise Execution Instance
               ↓
            EXECUTABLE
```

Replay uses the recorded Adapter/SpAd and Filter execution definition. It does not re-resolve a different Adapter, substitute a different SpAd, reorder Filters or replace recorded Filter configuration.

A different provider/model MAY be selected for replay when the recorded provider interface is preserved. If the recorded provider Adapter is a SpAd, the selected replay provider/model MUST also conform to that SpAd's declared provider/model or model-family applicability. For example, a provider change may be valid for the same applicable Qwen model family when the SpAd permits it, while selecting an unrelated model such as DeepSeek fails when the SEP requires a Qwen-specific SpAd.

> **Replay provider/model selection remains provisional at this point in the contract. The exact path by which Repetere and/or Moderari selects a different replay provider/model, how Praebere participates in that selection, and how the selected target is supplied to SIE MUST be confirmed against the current implementation before this flow becomes an implementation invariant. Section 20.4 may therefore require refinement when that replay selection path is reviewed.**

The recorded SEP itself never transitions state and is never modified.

---

## 20A. SIE Filter Discovery

Moderari owns user-defined Filter Profiles, but SIE owns knowledge of the Filter implementations currently available to the interaction engine.

Moderari therefore requires a simple discovery facility through which it can ask SIE what Filters are available. This facility is analogous in purpose to Praebere supplying descriptive model/provider information to Moderari: it exposes SIE-owned capability information without transferring ownership of that capability to Moderari.

Conceptually:

```text
Moderari Core
      ↓
ask SIE for available Filter definitions
      ↓
     SIE
      ↓
Filter discovery / manifests
      ↓
descriptive Filter metadata
      ↓
Moderari Core
```

The initial metadata is expected to provide enough information for Moderari to identify and describe a Filter, including conceptually:

```text
FilterDescriptor
    identity
    name
    version
    description
    requires[]
```

`identity` is the stable Filter identity used by Moderari when referring to the Filter. `name` and `description` are descriptive/user-facing metadata. `requires[]` exposes the Filter's declared model-characteristic requirements.

This discovery operation is **not session applicability evaluation**.

```text
SIE Filter discovery
    → what Filter implementations are available to Moderari?

configure_provider(session_id, emd)
    → which available Filters are applicable to this provider/model?
```

The discovery facility does not require a session, selected provider/model, or EMD.

The exact Filter metadata contract is intentionally not frozen by IP-01. Filter Profile creation may later require additional information such as configuration definitions, defaults, validation metadata, parameter types, or UI hints.

> **SIE MUST provide Moderari with a discovery facility through which Moderari can obtain descriptive metadata for the Filters currently available to SIE. The initial metadata is expected to include Filter identity, name, version, a short description and declared `requires[]`. The discovery contract may be extended when Filter Profile creation is implemented.**

This preserves the ownership boundary:

```text
Praebere
    → describes available model/provider capability

SIE
    → describes available interaction-processing capability

Moderari
    → uses those descriptions to construct user-facing choices
```

---

## 21. Conceptual Moderari ↔ SIE Operations

The exact Python/API signatures should be derived from the existing implementation. The following operations describe the architectural conversation between Moderari and SIE.

### 21.1 Enabled-session construction

```text
create_session(session_id)

configure_provider(session_id, emd)
    -> ProviderConfigurationResult

configure_filters(session_id, ordered_filters)

set_client_interface(session_id, canonical_interface)
    -> AdapterStatusResult

finalise_execution_plan(session_id)
    -> immutable SessionExecutionPlan

# Moderari registers the immutable SEP with Vestigare.
# Only after Vestigare accepts:

materialise_execution(session_id)
    -> ExecutionInstance ready
```

`create_session(session_id)` creates temporary SIE construction state for a Moderari-owned session that may use SIE. It does not create the overall Lumen session, a SEP, or an Execution Instance.

`configure_provider(session_id, emd)` configures the provider execution boundary. SIE reads the Praebere-owned provider identity, model identity/family, provider interface and model attributes from the EMD. It resolves the effective **provider Adapter**, which may be the Base Adapter for the provider interface or an applicable Specialised Adapter (SpAd).

A SpAd is a **provider-boundary Adapter**. It specialises a Base Adapter/interface for known provider-, model-, model-family-, or provider/model-specific representation behaviour. `SpAd.IN` canonicalises provider/model representation to CIM; `SpAd.OUT` materialises CIM to the provider/model representation. SpAds are not resolved on the client boundary.

Conceptually:

```text
ProviderConfigurationResult
    adapter_status
        ADAPTER_ENABLED
        ADAPTER_DISABLED

    available_filters[]
```

If the provider Adapter is unavailable, `adapter_status = ADAPTER_DISABLED` and `available_filters = []`. SIE construction ends and Moderari creates the `ADAPTERS_DISABLED` SEP. If the provider Adapter is available, SIE evaluates Filter applicability against the model attributes/requirements and returns the available Filters to Moderari. This availability set is discovery output and need not be retained by SIE.

`configure_filters(session_id, ordered_filters)` receives Moderari's resolved user selection. Moderari owns Filter Profiles and converts the selected Profile to exact ordered Filter identities, versions and runtime configuration. `ordered_filters = []` represents the explicit `Filters: None` selection. SIE retains the selected Filter execution definition and preserves its order exactly.

`set_client_interface(session_id, canonical_interface)` is called after the first Ask reaches Moderari. Moderari identifies the client interface using the canonical SIE interface identity while holding the Ask. SIE resolves the **Base client Adapter** for that interface and returns:

```text
AdapterStatus
    ADAPTER_ENABLED
    ADAPTER_DISABLED

AdapterStatusReason
    ADAPTER_NOT_AVAILABLE
    ADAPTER_NOT_REQUIRED
```

SIE uses the complete construction state to return one of three effective outcomes:

```text
ADAPTER_ENABLED
    → finalise_execution_plan()
    → ADAPTERS_ENABLED SEP

ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
    → ERROR
    → end session

ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
    → discard SIE construction
    → Moderari creates ADAPTERS_DISABLED SEP
    → direct pass-through
```

`ADAPTER_NOT_AVAILABLE` means SIE work is required but the required client Base Adapter cannot be resolved. `ADAPTER_NOT_REQUIRED` means the client and provider interfaces are the same, no provider SpAd is required, and no Filters were selected, so SIE has no work to perform.

`finalise_execution_plan(session_id)` is called only when the provider Adapter, selected ordered Filters, client interface and client Base Adapter have been established and the complete enabled execution path validates. It creates and returns the final immutable `SessionExecutionPlan`. No Execution Instance is created by this operation.

Moderari then registers that immutable SEP with Vestigare. If Vestigare rejects or fails, Moderari terminates the session and instructs SIE to destroy the active SEP/session state.

`materialise_execution(session_id)` is called only after Vestigare has accepted the immutable SEP. SIE then materialises the runtime `ExecutionInstance`. Successful materialisation allows Moderari to transition the session to `READY` and release the held first Ask.

### 21.3 Normal enabled execution

```text
process_request(session_id, external_request)
process_response(session_id, external_response)
```

These operations execute the already materialised immutable Plan. No Adapter, SpAd or Filter resolution occurs per Ask.

### 21.4 Replay — provisional pending replay-path inspection

Replay follows a separate establishment path rather than the normal enabled-session construction sequence.

```text
Repetere
    ↓
\\obt carrying recorded immutable SEP
    ↓
Moderari
    ↓
selected replay provider/model established
    ↓
Moderari → SIE:
    recorded immutable SEP
    +
    replay provider/model EMD
    ↓
SIE validates recorded provider-side execution
    ↓
ACCEPT / DECLINE
```

Conceptually:

```text
validate_replay(recorded_sep, replay_emd)
    -> ACCEPT | DECLINE
```

The recorded client boundary is already defined by the SEP and is not re-resolved. SIE validates the selected replay provider/model against the recorded provider interface and recorded provider Adapter. If that Adapter is a SpAd, the replay target must satisfy the recorded SpAd's provider/model or model-family applicability. SIE does not resolve or substitute a different SpAd.

Validation and replay materialisation remain separate concerns. The exact replay materialisation operation and the exact path by which Repetere/Moderari/Praebere selects and supplies an alternative replay provider/model MUST be confirmed against the current implementation before the replay API becomes normative.

### 21.5 Lifecycle

```text
end_session(session_id)
```

`end_session` destroys the transient SIE construction/runtime state associated with the active session, including any Execution Instance. If a final SEP exists only as active SIE session state because Vestigare registration failed, that active reference is also discarded. Historical SEP evidence already accepted by Vestigare is not altered.

---

## 22. Failure and Outcome Rules

The contract distinguishes **normal SEP construction outcomes**, **replay validation failures**, and **runtime execution failures**. These are not the same class of failure and MUST NOT be conflated.

### 22.1 Normal session construction

Normal establishment determines whether SIE has an enabled execution responsibility.

Provider-side Adapter resolution occurs first:

```text
provider Adapter unavailable
    → SIE construction ends
    → no Filters offered
    → Moderari creates ADAPTERS_DISABLED SEP
    → direct provider pass-through
```

This is not an establishment error. Any incompatibility encountered by the direct provider interaction is outside SIE and is returned through the normal provider path.

If a provider Adapter exists, applicable Filters may be offered and the user may select a Filter Profile or `Filters: None`. When the first Ask establishes the client interface, SIE makes the final enabled/disabled decision:

```text
ADAPTER_ENABLED
    → complete SIE path exists and SIE has work to perform
    → finalise ADAPTERS_ENABLED SEP

ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE
    → SIE work is required but client Base Adapter is unavailable
    → ERROR
    → end session

ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED
    → client interface == provider interface
    → provider requires no SpAd
    → selected_filters == []
    → no SIE work is required
    → discard SIE construction
    → Moderari creates ADAPTERS_DISABLED SEP
    → direct pass-through
```

A provider SpAd or any selected Filter means SIE has actual work to perform even when the client and provider interfaces are identical.

> **SIE finalises an `ADAPTERS_ENABLED` SEP only when the required client and provider Adapter boundaries exist and SIE has representation-correction or CIM-processing work to perform.**

Invalid Filter configuration supplied during construction is rejected before SEP finalisation. It does not create an invalid SEP.

SEP finalisation and runtime materialisation remain separate. No Execution Instance exists for either `ADAPTERS_DISABLED` outcome.

### 22.2 Replay validation

Replay is stricter because the immutable SEP already records the historical execution definition.

SIE MUST return `DECLINE` when faithful replay cannot be established, including when:

- the recorded CIM version is unavailable or incompatible;
- the recorded client Adapter identity/version is unavailable;
- the recorded provider Adapter identity/version is unavailable;
- a recorded SpAd's applicability is not satisfied by the selected replay provider/model;
- a recorded Filter identity/version is unavailable;
- the recorded Filter configuration cannot be validated or reconstructed faithfully;
- the recorded execution definition cannot be materialised faithfully; or
- faithful execution would require component substitution, re-resolution, reordering, version replacement, CIM upgrade/downgrade, or mutation of the recorded SEP.

Substitution and re-resolution are **not recovery mechanisms** for replay. If the recorded execution definition cannot be executed faithfully, SIE returns `DECLINE`.

### 22.3 Runtime execution failures

Failures that occur while executing an already valid and materialised SEP are execution failures, not SEP-construction failures.

In particular:

```text
private provider response
      ↓
Provider Adapter.IN
      ↓
CIM + filter_execution_id
      ↓
reverse Filter traversal
      ↓
identifier remains unclaimed
      ↓
EXECUTION FAILURE
```

A private `filter_execution_id` that reaches the end of reverse Filter traversal unclaimed MUST cause that execution to fail and MUST NOT escape through the client Adapter.

Other runtime component or provider failures are handled as execution failures according to their execution boundary; they do not retroactively invalidate or mutate the immutable SEP.

### 22.4 SEP immutability is structural

Attempted SEP mutation is not defined as a runtime failure case because the contract exposes no operation that permits mutation.

> **A finalised `SessionExecutionPlan` is immutable by construction. There is no Moderari ↔ SIE operation that modifies an existing final SEP.**

A changed execution definition requires construction of a new Plan/session execution definition rather than mutation of the existing SEP.

---

## 23. Vestigare Evidence Relationship

The SEP is supplied once during establishment.

```text
SESSION ESTABLISHMENT

session_id
    ↓
immutable Session Execution Plan ─────────→ Vestigare
    │
    └── exact execution definition

INTERACTIONS

session_id
    ↓
Ask / Response evidence ──────────────────→ Vestigare
    │
    └── interpreted against registered SEP
```

The resolved ordered Filter list and exact configuration are authoritative execution provenance. A Filter Profile name alone is not sufficient because Profiles are mutable user-facing configuration resources.

---

## 24. Session End and Restart Behaviour

For enabled sessions, ending the Moderari session destroys the transient SIE construction/runtime state and Execution Instance associated with that active session.

The immutable SEP already registered with Vestigare remains historical evidence.

The current architectural direction remains **no live session recovery across Moderari shutdown/restart**. This must be confirmed against the current Moderari shutdown/session implementation before becoming an implementation invariant.

Historical execution reconstruction is performed through Repetere using the recorded SEP, not by restoring an active SIE session.

---

## 25. Normative Rules

1. **Moderari owns the Session Execution Plan.**
2. **Every executable Moderari session has an immutable SEP.**
3. **`ADAPTERS_DISABLED` is a complete SEP and causes no Moderari ↔ SIE communication.**
4. **SIE creates, validates, finalises, materialises and executes `ADAPTERS_ENABLED` Plans.**
5. **A `SessionExecutionPlan` exists only in finalised immutable form.**
6. **Moderari MUST NOT alter a Plan returned by SIE.**
7. **Praebere-owned provider/model/interface facts are carried by the EMD.**
8. **Adapters are client/provider boundary identities; IN/OUT are operations selected by flow direction.**
9. **SIE is flow control: each action output becomes the next action input.**
10. **Filters consume and emit CIM.**
11. **Moderari owns Filter Profiles and supplies the resolved ordered Filter configuration.**
12. **SIE preserves Filter order exactly and traverses it forward to-model and reverse from-model.**
13. **The client interface is established once from the first Ask using the canonical interface enum and cannot change during the session.**
14. **A private Filter creates its own opaque `filter_execution_id`; SIE need not maintain an ID-to-Filter ownership map.**
15. **A CIM carrying `filter_execution_id` MUST be claimed by exactly one Filter; an unclaimed identifier causes execution failure and MUST NOT escape through the client Adapter.**
16. **Moderari owns the overall session `READY` state.**
17. **Vestigare MUST successfully receive the immutable SEP before Moderari marks the session `READY` or releases the first Ask for provider execution.**
18. **For replay, Repetere supplies the entire recorded SEP to Moderari.**
19. **A recorded enabled SEP bypasses component resolution but MUST pass exact identity/version validation before materialisation.**
20. **Replay MUST NOT substitute, upgrade, downgrade, reorder, re-resolve or mutate the recorded Plan.**
21. **A SpAd is a provider-boundary Adapter; the client boundary resolves only the Base Adapter for the canonical client interface.**
22. **A SpAd may specialise provider-, model-, model-family-, or provider/model-specific representation behaviour and may override provider-boundary `IN`, `OUT`, or both.**
23. **SEP finalisation precedes Vestigare registration; Execution Instance materialisation occurs only after Vestigare accepts the immutable SEP.**
24. **The availability of Adapters alone does not require `ADAPTERS_ENABLED`; SIE must have actual representation-correction or CIM-processing work to perform.**
25. **`ADAPTER_DISABLED / ADAPTER_NOT_AVAILABLE` at the client boundary is an error and ends the session.**
26. **`ADAPTER_DISABLED / ADAPTER_NOT_REQUIRED` causes SIE to discard construction state; Moderari creates the `ADAPTERS_DISABLED` SEP and uses direct pass-through.**
27. **`ADAPTER_NOT_REQUIRED` applies when client and provider interfaces are the same, no provider SpAd is required, and no Filters are selected.**
28. **SIE exposes descriptive metadata for its available Filters to Moderari; Filter discovery is independent of session-specific Filter applicability evaluation.**

---

## 26. Implementation Boundary for the Next Milestone

The next SIE foundation milestone should turn this architecture into exact code contracts and tests before full request/response transformation execution is implemented.

Its scope should include:

- concrete immutable SEP types and execution-mode discriminator;
- enabled SIE construction-state types;
- EMD input contract using the actual current Praebere schema;
- provider Adapter resolution and candidate Filter availability result types;
- ordered Filter selection/configuration types and validation;
- canonical client-interface establishment;
- client/provider Adapter references rather than permanent IN/OUT Adapter identities;
- enabled SEP finalisation rules;
- Execution Instance materialisation from a final SEP;
- exact recorded-SEP validation/materialisation entry point for replay;
- Moderari/SIE state-transition contracts;
- disabled-path SEP contract with no SIE involvement;
- mandatory Vestigare registration gate before Moderari `READY`;
- private Filter `filter_execution_id` reverse-traversal contract; and
- failure/result types for plan construction, materialisation and replay validation.

Before implementation, the current Moderari source MUST be inspected for:

- existing session state/lifecycle names;
- first-Ask handling and client-interface recognition;
- shutdown/session cleanup behaviour;
- any existing Adapter enable/disable setting and routing path; and
- current Praebere EMD structure.

The implementation should reuse existing canonical interface enums and production component catalogue types rather than introducing parallel representations.

---

## 27. Governing Statement

> **Moderari owns the session and its immutable Session Execution Plan. When Adapters are disabled, Moderari records and registers that direct execution Plan without involving SIE. When Adapters are enabled, SIE constructs, validates, finalises, materialises and executes the Plan. SIE is the flow-control gateway: client Adapter IN → ordered CIM Filters → provider Adapter OUT, and provider Adapter IN → reverse ordered CIM Filters → client Adapter OUT. The frozen Plan is registered with Vestigare before Moderari declares the session READY, and the same immutable Plan is later sufficient for Repetere to request exact, version-validated replay without re-resolving the historical execution environment.**
