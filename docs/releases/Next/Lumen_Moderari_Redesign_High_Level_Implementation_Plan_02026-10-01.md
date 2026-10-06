# Lumen Moderari Redesign --- High-Level Implementation Plan

**Date:** 02026-10-01\
**Status:** Planning baseline --- to be refined after architecture
review\
**Scope:** Praebere, Pontis, Moderari, Rogare, Vestigare and Repetere
changes required to implement the current Moderari redesign

------------------------------------------------------------------------

# 1. Purpose

This document provides a high-level implementation sequence for the
current Lumen Moderari redesign.

It deliberately does **not** attempt to turn the architecture into
detailed coding tasks yet. The architecture documents should first
complete review and become the implementation baseline. Detailed service
plans, migration steps and acceptance tests can then be produced against
the actual code.

The proposed implementation begins with **Praebere** because Praebere
supplies the provider/model truth required by the other redesigned
services.

The overall dependency direction is:

``` text
Praebere
    │
    │ provider/model truth
    ▼
Pontis
    │
    │ session + client boundary
    ▼
Moderari
    │
    │ compatibility + intentional transformation
    ▼
Rogare / Vestigare / Repetere integration
```

This is an implementation order, not a runtime call hierarchy.

------------------------------------------------------------------------

# 2. Implementation Principles

The implementation should preserve the following architectural
boundaries.

``` text
Praebere Model Profile
    = user-maintained model knowledge

Effective Model Definition
    = Praebere-resolved provider/model truth

Moderari Adapter
    = technical compatibility

Moderari Filter
    = executable intentional transformation

Moderari Filter Profile
    = user-owned configuration convenience

Vestigare Raw Trace
    = what was actually said

Vestigare Execution Metadata
    = environment under which it occurred
```

Additional governing principles:

> **Praebere remains the authority for provider/model knowledge.**

> **Pontis owns session lifetime.**

> **Moderari performs compatibility and intentional interaction
> transformation; it does not become a second provider/model
> authority.**

> **Published interaction schemas define base representation Adapters.
> Provider/model-specific differences belong to specialised Adapters.**

> **The CIM belongs only to the enabled Adapter framework.**

> **Filter transformations do not become part of the Vestigare raw
> conversation trace.**

> **Unknown does not mean broken. Unknown means do not interfere unless
> compatibility can be established safely.**

> **Continuous New-Code Ask Path:** During implementation, Lumen should
> prioritise maintaining an end-to-end Ask path that exercises the newly
> implemented architecture. When a service change breaks an existing
> downstream contract, make the smallest appropriate dependent-service change
> required to consume the new contract and restore the Ask path. Do not retain
> obsolete behaviour solely to prove that the old path still works.

Maintaining the Ask path is a priority, not an absolute requirement. Where
restoring it would require disproportionate, premature or throwaway work, the
break may be accepted temporarily. Such breakpoints must be explicit, bounded,
and the Ask path should be restored at the earliest useful implementation
point.

------------------------------------------------------------------------

# 3. Implementation Strategy

The redesign should be implemented incrementally rather than replacing
all services simultaneously.

Each stage should leave the Stack in a testable state.

A useful progression is:

``` text
1. Praebere authority
        ↓
2. Pontis session boundary
        ↓
3. Vanilla Moderari
        ↓
4. Adapter Framework + CIM
        ↓
5. Qwen specialised Adapter
        ↓
6. Filter Framework
        ↓
7. System Prompt Filter
        ↓
8. Existing Context/Cognition investigation
        ↓
9. Rogare configuration workflow
        ↓
10. Vestigare evidence integration
        ↓
11. Repetere integration
        ↓
12. End-to-end migration and acceptance
```

The phases identify the **primary implementation focus**, not rigid
service-by-service isolation. A change in the current primary service may
require a small change in another service so that the new contract can be
exercised through a real Ask.

The preferred rhythm is:

``` text
implement new behaviour
        │
        ▼
make minimum required dependent-service changes
        │
        ▼
run end-to-end Ask
        │
        ├── prove the new behaviour was exercised
        └── verify the Stack remains operational
        │
        ▼
continue
```

The purpose of preserving the Ask path is therefore **not** to demonstrate that
the old implementation still works. It is to obtain early integration evidence
that the newly implemented architecture works through the real Stack.

------------------------------------------------------------------------

# 4. Phase 0 --- Freeze the Architecture Baseline

Before code changes begin:

1.  Review the Moderari Redesign Proposal.
2.  Review the Adapter Architecture.
3.  Review the Canonical Interaction Model Architecture.
4.  Review the Filter Architecture.
5.  Review the companion service changes.
6.  Resolve only architecture questions that block implementation.
7.  Explicitly mark remaining implementation choices as implementation
    decisions rather than continuing to expand the architecture.

The reviewed documents become the implementation baseline.

Do not attempt to settle the detailed Context/Cognition design during
this phase. Its behaviour is deliberately deferred until the existing
implementation is inspected.

**Exit condition:** the service boundaries and contracts required to
start Praebere are stable enough to code.

------------------------------------------------------------------------

# 5. Phase 1 --- Praebere First

Praebere should be implemented first because Moderari, Pontis and Rogare
depend on authoritative provider/model information.

## 5.1 Replace Global Model Selection with Session Reservations

Move from the existing runtime-global selected-model concept toward:

``` text
session_id
    │
    └── provider
          │
          └── model
```

Praebere owns this reservation.

Required lifecycle:

``` text
Pontis creates session
        │
Rogare selects model
        │
        ▼
Praebere
    reserve model for session
        │
first Ask
        ▼
activate execution
        │
session end
        ▼
release reservation / residency
```

## 5.2 Establish Provider Representation Knowledge

Praebere must expose the external interaction representation used by the
provider interface.

For example:

``` yaml
provider:
  id: ollama
  interaction_representation: openai-chat-completions
```

This describes the provider API boundary, not the model's internal
prompt/token representation.

## 5.3 Establish Effective Model Definition

Define the minimum Effective Model Definition required by current
consumers.

It should combine only authoritative information required by Lumen,
potentially from:

``` text
provider discovery
+
Lumen catalogue
+
user Model Profile
+
verification
=
Effective Model Definition
```

Preserve provenance where required.

Do not move Adapter knowledge into Praebere.

## 5.4 Session Resolution API

Provide the operation Moderari will use on first Ask to obtain the
authoritative execution definition for a `session_id`.

The exact API shape is an implementation decision, but its ownership is
not:

> **Moderari asks Praebere what is running; Moderari does not
> reconstruct the answer itself.**

## 5.5 Activation and Release

Retain/refine the existing activation responsibility:

``` text
activate execution(session_id)
release execution(session_id)
```

Praebere remains responsible for provider/model readiness and model
residency.

## 5.6 Exercise New Praebere Behaviour Through the Stack

Do not preserve the old Praebere contract merely so existing Pontis or Moderari
code continues to work.

As each significant Praebere contract is introduced, determine whether the
current consumers can exercise it through a real Ask. If not, make the minimum
appropriate Pontis and/or Moderari change necessary to consume the **new**
Praebere contract.

For example:

``` text
new Praebere session/model behaviour
        │
        ▼
minimum Pontis/Moderari consumer change
        │
        ▼
real Ask through Stack
        │
        ├── new Praebere path exercised
        ├── new messages/contracts exercised
        ├── selected model actually invoked
        └── response returned
```

These dependent changes do not change the primary focus of Phase 1. They exist
to prove Praebere's new behaviour in its real execution path.

If restoring the Ask path would require substantial implementation belonging
properly to a later phase, record the temporary break explicitly and restore
the path as soon as the next useful dependency is implemented.

**Phase 1 exit condition:** a session can reserve a model, resolve its
Effective Model Definition/provider representation, activate it, and
release it without another service maintaining competing model truth.

------------------------------------------------------------------------

# 6. Phase 2 --- Pontis Session and Boundary Changes

Once Praebere's session model is stable, update Pontis.

## 6.1 Remove Model Authority

Remove remaining Pontis behaviour that makes it authoritative for the
model or rewrites requests from a locally held authoritative model
value.

Retain:

``` text
session creation
session lifetime
request/response routing
execution activation coordination
external \obt forwarding
```

## 6.2 Retain Execution Activation Tracking

Pontis may remember that execution activation has occurred:

``` text
_active_execution_sessions
```

but Praebere remembers **what** was activated.

> **Pontis remembers that activation happened. Praebere remembers what
> was activated.**

## 6.3 Establish Client Representation

Pontis records the client interaction representation from the ingress
contract.

For example:

``` text
ACP ingress
    -> client representation = ACP

OpenAI-compatible ingress
    -> client representation = OpenAI Chat Completions
```

No payload sniffing is required.

## 6.4 Session-End Notification

Ensure Pontis reliably announces session end to every service holding
session-scoped state, initially including:

``` text
Praebere
Moderari
```

**Phase 2 exit condition:** Pontis owns session lifecycle and
client-boundary facts without owning provider/model truth.

------------------------------------------------------------------------

# 7. Phase 3 --- Establish Vanilla Moderari

Before implementing new Filters or broad protocol translation, reduce
Moderari to the new architectural baseline.

Target:

``` text
Pontis
   │
   ▼
Moderari
   │
   ▼
Provider / Model
```

Vanilla Moderari should:

-   maintain required session conversation/model-facing context for the
    current scope;
-   obtain authoritative execution information from Praebere on first
    Ask;
-   materialise immutable Moderari Session State;
-   support zero Filters;
-   support Adapter processing Enabled/Disabled;
-   avoid provider discovery/model lifecycle responsibilities;
-   contain no default System Prompt;
-   clean up session-scoped live state when Pontis ends the session.

At this stage, preserve existing behaviour only where needed to keep the
path operational.

**Phase 3 exit condition:** a no-Filter, no-new-Adapter interaction can
pass through the redesigned Moderari lifecycle.

------------------------------------------------------------------------

# 8. Phase 4 --- Adapter Framework and CIM

Implement the compatibility framework independently of Filter behaviour.

## 8.1 Extension Discovery

Implement restart-time discovery from the configured extension location.

Conceptually:

``` text
/lumen/extensions/adapters/
        │
        ▼
Moderari Adapter Discovery
```

Separate:

``` text
discovered
trusted/enabled
applicable
selected
```

## 8.2 Base Representation Adapter Contract

Define the executable contract for a base representation Adapter.

A base Adapter implements a **published external representation
contract** and maps:

``` text
Published External Representation
              ⇅
             CIM
```

The initial design work should use the published OpenAI and Anthropic
interaction schemas as the first concrete representation contracts.

This does not require implementing every protocol Adapter immediately.

## 8.3 CIM 1.0 Minimum Schema

Implement only the semantic concepts required by the first real
interoperability cases.

Likely initial concepts include:

``` text
message
system context
tool definition
tool call
tool result
```

Do not attempt to model every known AI protocol feature.

## 8.4 Specialised Adapter Contract

Provide a mechanism by which a provider/model-specific Adapter can
specialise a base representation Adapter and supply only required
differences.

Conceptually:

``` text
OpenAI Adapter
      │
      └── Qwen/OpenAI Specialisation
              overrides only deviations
```

The architecture does **not** currently mandate whether this is
implemented through:

``` text
inheritance
delegation
composition
override hooks
another extension mechanism
```

Choose that mechanism during implementation design.

There is no model-correction Adapter chain.

## 8.5 Loss Boundary

Keep two cases separate:

``` text
Published representation A
    -> CIM
    -> Published representation B

Loss here:
    Lumen interoperability concern
```

versus:

``` text
Provider/model deviation
    -> specialised Adapter override
    -> base representation semantics

Loss here:
    specialised Adapter author's responsibility
```

**Phase 4 exit condition:** Moderari can discover representation
Adapters, resolve an enabled compatibility path, create transient CIM
instances where required, and preserve pass-through behaviour when no
translation is required.

------------------------------------------------------------------------

# 9. Phase 5 --- Migrate the Existing Qwen Translator

Use the current Qwen tool-call translator as the first specialised
Adapter implementation.

The objective is not to redesign Qwen behaviour.

The objective is to prove:

``` text
existing Qwen-specific compatibility behaviour
                │
                ▼
new Adapter extension contract
```

The Qwen Adapter should contain only the provider/model-specific
differences required on top of its base representation behaviour.

No Qwen-specific conditional should remain in Moderari Core.

This phase is an important architectural test: if migrating Qwen
requires modifying Moderari Core with Qwen knowledge, the extension
boundary is incomplete.

**Phase 5 exit condition:** existing Qwen compatibility works through
the generic Adapter Framework.

------------------------------------------------------------------------

# 10. Phase 6 --- Filter Framework

With the technical compatibility path stable, implement the independent
Filter extension mechanism.

Required initial capabilities:

``` text
discovery
self-description
requirements
configuration schema
session-scoped resolution
execution
version/provenance
transformation evidence
```

Filters operate on Moderari-managed textual/contextual material, not
CIM.

A Filter Profile remains configuration-time convenience:

``` text
Filter Profile
      │
      ▼
resolved Filter set
      │
      ▼
session execution truth
```

Explicit `None` resolves to zero Filters.

**Phase 6 exit condition:** Moderari can resolve and execute an empty or
populated Filter set without Filter-specific logic in Core.

------------------------------------------------------------------------

# 11. Phase 7 --- System Prompt Filter

Move existing System Prompt behaviour into the first concrete Filter.

Remove the Moderari default System Prompt.

Support:

``` text
No System Prompt Filter
    -> no Lumen-added system prompt

System Prompt Filter
    -> configured system context

Custom System Prompt
    -> Filter Profile containing System Prompt Filter
```

Saved System Prompts remain configuration resources rather than
execution identity.

**Phase 7 exit condition:** all deliberate System Prompt behaviour
occurs through the Filter Framework.

------------------------------------------------------------------------

# 12. Phase 8 --- Investigate Existing Context/Compaction Behaviour

Do **not** implement the proposed Context/Cognition Filter directly from
the architecture discussion.

First inspect and test the existing Moderari implementation.

Determine:

``` text
what triggers compaction
what context exists immediately before it
what private model request is made
what response is received
what context is replaced
what context is retained
what the client subsequently sees
what Vestigare currently records
what happens on later Asks
```

Create targeted tests around the old behaviour before replacing it.

Only after this evidence exists should the Context/Cognition Filter
design be finalised.

The likely architectural direction remains:

``` text
Moderari context
      │
      ▼
Context/Cognition Filter
      │
      ├── may perform private inference
      │
      └── produces replacement model-facing context
```

but its exact semantics remain deliberately open.

**Phase 8 exit condition:** existing behaviour is understood well enough
to write the Context/Cognition implementation specification.

------------------------------------------------------------------------

# 13. Phase 9 --- Rogare Session Configuration Workflow

Update Rogare after the underlying service contracts exist.

Target lifecycle:

``` text
Start Session
      │
      ▼
Pontis session created
      │
Select Model
      │
      ▼
Praebere reservation
      │
Select Filter Profile
      │
      ├── named profile
      └── explicit None
      │
      ▼
Interaction Ready
```

The user-context input and Ask action remain disabled until:

``` text
session exists
AND
model selected
AND
Filter Profile decision made
```

Adapter selection is not exposed to the user.

The user control remains:

``` text
Adapter processing
    Enabled / Disabled
```

Moderari determines the applicable Adapter implementation.

**Phase 9 exit condition:** Rogare configures a session according to the
new ownership boundaries without duplicating compatibility/model logic.

------------------------------------------------------------------------

# 14. Phase 10 --- Vestigare Evidence Integration

Preserve the evidence boundary.

## 14.1 Raw Trace

Vestigare records:

``` text
exact client interaction
exact raw provider/model response
```

Filter-transformed context does not replace raw trace content.

## 14.2 Execution Metadata

At Moderari Session State materialisation, record relevant environment
metadata such as:

``` text
provider/model identity
Effective Model Definition provenance/revision
client/provider representations
effective Adapter identity/version/state
resolved Filters
Filter versions/configuration
```

## 14.3 Moderari Transformation Evidence

Filter execution/transformation records remain separate from raw
Vestigare conversation content.

The implementation should preserve:

> **Vestigare records what was said. Moderari records what it did.**

**Phase 10 exit condition:** a recorded interaction can distinguish raw
conversation, execution environment and Moderari transformation
evidence.

------------------------------------------------------------------------

# 15. Phase 11 --- Repetere Integration

Once the execution/evidence model is stable, update Repetere.

The raw Vestigare trace remains replay source material.

``` text
Vestigare Raw Trace
        │
        ▼
Repetere
        │
        ▼
new Lumen session/execution
```

Repetere should establish the required Moderari environment rather than
depending on Filter-transformed content having been embedded into the
historical conversation.

Detailed replay variation controls remain outside this initial
implementation unless required to restore current functionality.

**Phase 11 exit condition:** a raw recorded interaction can be replayed
through the redesigned session/configuration lifecycle.

------------------------------------------------------------------------

# 16. Phase 12 --- End-to-End Interoperability

Once the individual service changes are stable, exercise the complete
architecture.

Initial paths should include:

``` text
Rogare/Pi
    -> Pontis
    -> Moderari
    -> Ollama
    -> Qwen
```

and the existing Colibri/provider work where appropriate.

Then test the CIM's broader purpose with at least one genuine
representation bridge when its Adapter exists:

``` text
OpenAI-representation client
        │
        ▼
OpenAI Adapter
        │
       CIM
        │
        ▼
Anthropic Adapter
        │
        ▼
Anthropic-representation provider
```

This validates the developer-facing portability property:

> **An application can continue using the interaction representation it
> was written for while Lumen translates to the representation required
> by another provider, where the semantics are interoperable.**

------------------------------------------------------------------------

# 17. Migration Approach

Avoid both a single cut-over and compatibility work whose only purpose is to
keep obsolete behaviour alive.

The preferred migration unit is a **small vertical architectural change**:

``` text
change authoritative service
        │
        ▼
introduce new contract/behaviour
        │
        ▼
change only the dependent code required to consume it
        │
        ▼
run a real Ask through the Stack
        │
        ▼
prove the NEW path
        │
        ▼
continue / remove superseded code when safe
```

A working Ask is therefore an integration checkpoint, not a backwards-
compatibility objective.

For every meaningful implementation step, the detailed plan should identify:

``` text
starting runnable state
new behaviour being introduced
new or changed contract/messages
minimum dependent-service changes
whether an end-to-end Ask should work
what new behaviour that Ask proves
resulting runnable state
obsolete code now eligible for removal
```

## 17.1 Priority, Not Absolute Constraint

The Continuous New-Code Ask Path is a priority.

There will be cases where restoring an Ask immediately would require:

``` text
disproportionate work
premature implementation of a later subsystem
throwaway compatibility code
or an unsafe partial contract
```

In those cases, do not distort the architecture merely to keep the Stack
green.

Instead:

``` text
declare temporary Ask break
        │
        ▼
identify exact blocking contract/change
        │
        ▼
make the smallest coherent set of changes
        │
        ▼
restore Ask
        │
        ▼
exercise the new path before unrelated work continues
```

Temporary break windows should therefore be deliberate and bounded.

## 17.2 No Old-Path Success Criterion

Do not count this as sufficient validation:

``` text
new service code exists
        │
old compatibility path still handles execution
        │
Ask succeeds
```

The successful Ask must exercise the newly introduced behaviour that the
implementation step is intended to prove.

For example, after changing Praebere's session/model contract, the meaningful
test is not that an old Praebere interface can still produce a response. The
meaningful test is that Pontis/Moderari consume the new Praebere contract and a
real Ask reaches the intended provider/model through it.

## 17.3 Staged Removal

Particularly important candidates for staged removal remain:

``` text
Praebere global selected model
Pontis model authority/request rewriting
Moderari embedded Qwen conditions
Moderari default System Prompt
old System Prompt policy machinery
old Qwen translator wiring
old compaction machinery
```

Remove obsolete paths when their replacement has been exercised sufficiently
through the new vertical path.

Do not remove the old compaction machinery until Phase 8 has established what
it actually does.

------------------------------------------------------------------------

# 18. Testing Strategy

Testing should occur at four levels.

The fourth level — the Continuous New-Code Ask test — is especially important
during this redesign because responsibilities are moving between services.

## 18.1 Service Contract Tests

Verify ownership boundaries independently.

Examples:

``` text
Praebere resolves session -> provider/model
Pontis establishes client representation
Moderari materialises immutable session state
Adapter discovery/resolution is generic
Filter discovery/resolution is generic
session end releases service state
```

## 18.2 Behavioural Regression Tests

Preserve required existing behaviour during migration, especially:

``` text
Qwen tool calls
System Prompt behaviour
conversation continuity
session cleanup
provider activation/release
Vestigare recording
replay
```

## 18.3 Continuous New-Code Ask Tests

After each meaningful cross-service change, attempt an end-to-end Ask through
the Stack.

The test must identify what **new** behaviour is being exercised.

A useful checkpoint record is:

``` text
Ask path: PASS / TEMPORARILY UNAVAILABLE

New behaviour exercised:
    <specific new contract or implementation>

Dependent changes required:
    <services/modules changed to consume it>

Observed result:
    <provider/model reached, response returned, state/evidence checked>
```

A successful response alone is insufficient if execution silently used an old
compatibility path.

Where the Ask cannot reasonably be restored at that point, record why, what
change is blocking it, and the implementation point at which it is expected to
return.

## 18.4 Architectural Acceptance Tests

Tests should prove absence of forbidden coupling as well as successful
behaviour.

Examples:

``` text
No Qwen-specific branch in Moderari Core
No Adapter identity knowledge in Praebere
No model authority in Pontis
No protocol knowledge in Filters
No Filter-transformed context in Vestigare raw trace
No CIM path when Adapters are Disabled
No provider/model quirk added to CIM
```

------------------------------------------------------------------------

# 19. Suggested Implementation Work Packages

After architecture review, create detailed implementation plans in
approximately this order:

``` text
IP-01  Praebere Session Model and Effective Model Definition
IP-02  Pontis Session/Representation Boundary
IP-03  Vanilla Moderari and Session State
IP-04  Moderari Adapter Extension Framework
IP-05  CIM 1.0
IP-06  Qwen Specialised Adapter Migration
IP-07  Moderari Filter Extension Framework
IP-08  System Prompt Filter Migration
IP-09  Existing Context/Compaction Investigation
IP-10  Context/Cognition Filter — only after IP-09
IP-11  Rogare Session Configuration
IP-12  Vestigare Execution Evidence Integration
IP-13  Repetere Integration
IP-14  End-to-End Interoperability and Migration Cleanup
```

Each work package should be produced from the then-current source code,
not from architecture alone.

Each work package must also identify its **Continuous New-Code Ask checkpoints**:
the points at which the newly implemented behaviour should be exercised through
the Stack, any minimum dependent-service changes required to make that possible,
and any deliberate interval during which an Ask is expected not to work.

------------------------------------------------------------------------

# 20. First Implementation Target

The recommended first implementation target is therefore:

> **Praebere: replace global model selection with authoritative
> per-session provider/model reservation and expose the Effective Model
> Definition/provider interaction representation required by the rest of
> Lumen.**

That gives the redesign a stable foundation.

Once Praebere can answer:

``` text
For session X:

Which provider?
Which model?
Which external interaction representation?
What model characteristics are known?
What is their provenance?
Is the execution target ready?
```

Pontis and Moderari can be changed against a concrete authority rather
than provisional assumptions.

------------------------------------------------------------------------

# 21. Definition of Completion

The redesign implementation is complete at the high level when:

1.  Praebere is the sole provider/model authority and operates per
    session.
2.  Pontis owns session lifetime and client ingress representation
    without owning model truth.
3.  Moderari materialises immutable session execution state from
    authoritative facts.
4.  Vanilla Moderari works with zero Filters.
5.  Adapter processing can be enabled or disabled.
6.  Published representation Adapters map through the CIM.
7.  Provider/model-specific behaviour is supplied through specialised
    Adapters rather than Core changes.
8.  The existing Qwen compatibility behaviour runs through that
    extension mechanism.
9.  Filters are independently discoverable and operate outside CIM.
10. System Prompt behaviour is a Filter.
11. Context/Cognition behaviour has been redesigned from observed
    existing behaviour rather than assumption.
12. Rogare follows the session → model → Filter Profile → ready
    lifecycle.
13. Vestigare keeps raw conversation separate from execution metadata
    and Filter transformation evidence.
14. Repetere can replay raw interactions through the redesigned
    execution environment.
15. Session-end cleanup releases all live session-scoped state.
16. A future representation Adapter can be added without changing
    Moderari Core, Praebere, Pontis, Rogare, Vestigare or Repetere for
    that specific representation.

------------------------------------------------------------------------

# 22. Immediate Next Step

After the architecture documents have been reviewed, begin with the
current Praebere source and produce **IP-01 --- Praebere Session Model
and Effective Model Definition**.

That implementation plan should map the architecture onto the actual
Praebere modules, data structures, endpoints, Nuntius commands and tests
before any code is changed.

IP-01 should also identify, step by step, the minimum Pontis and Moderari
consumer changes required to exercise each significant new Praebere contract
through a real Ask. Those changes should consume the new architecture rather
than preserve an obsolete Praebere execution path merely for compatibility.
