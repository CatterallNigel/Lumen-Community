# Lumen Post-M0.1 Engineering Planning

**Date:** 27 September 2026\
**Status:** Architectural planning\
**Scope:** Moderari, session-bound model ownership, and concurrent
Vestigare recording

------------------------------------------------------------------------

## Purpose

This document records the current planning discussion for the next phase
of Lumen engineering following the M0.1 Research Distribution.

M0.1 established a complete, distributable Lumen system and provides a
stable external research reference point. The next phase should
therefore avoid extending M0.1 itself and instead address architectural
assumptions that were reasonable while Lumen primarily managed one
active interaction path at a time.

Three areas are now closely related:

1.  Revisit the responsibility of **Moderari**, including context
    compaction.
2.  Move model selection from effectively runtime-global ownership to
    **session-bound Provider and Model ownership**.
3.  Extend **Vestigare** so that it can record evidence from multiple
    concurrent sessions.

These should not be treated as unrelated features. Together they
represent a transition from Lumen managing an interaction to Lumen
managing multiple independent execution sessions.

------------------------------------------------------------------------

# 1. Architectural Direction

The emerging execution model is:

``` text
Client
  ↓
Pontis
  ↓
Lumen Session Identity
  ↓
Moderari
  ↓
Provider Binding
  ↓
Model Binding
  ↓
Execution
  ↓
Response
```

In parallel:

``` text
Execution activity
  ↓
Vestigare
  ↓
Evidence
  ↓
Aestimare
```

The important separation is:

> **Vestigare records evidence. Aestimare interprets evidence.**

Vestigare should not need to understand the complete relationship
between agents, clients, providers or sessions in order to record what
occurred. Its responsibility is to preserve sufficient identity,
correlation and event evidence for those relationships to be
reconstructed later.

Aestimare can subsequently join those pieces of evidence and determine
what they mean.

------------------------------------------------------------------------

# 2. Why These Changes Belong Together

Per-session model ownership and concurrent evidence recording are two
sides of the same architectural change.

If Lumen allows:

``` text
Session A → Ollama  → qwen2.5-coder:14b
Session B → Ollama  → gemma3:4b
Session C → Colibri → qwen36-35b-a3b
Session D → another provider/model
```

then Vestigare must be able to record all four execution streams without
relying on a single concept of:

``` text
the current session
the selected model
the active trace
```

Similarly, Moderari must compose and manage context for the session
actually being executed rather than relying on process-wide assumptions.

The architectural unit therefore becomes the **execution session**.

A useful conceptual relationship is:

``` text
Session
  ├── Client identity / origin
  ├── Provider binding
  ├── Model binding
  ├── Moderari context state
  ├── Execution activity
  └── Vestigare evidence identity
```

This does not require every service to own all of this information. It
requires each service to receive and preserve the identity needed for
its own responsibility.

------------------------------------------------------------------------

# 3. Moderari --- Revisit the Responsibility

## Background

Moderari was effectively the first Lumen service, created before the
present service architecture existed.

Its original responsibilities developed when continuity, context
management and model interaction were much more tightly coupled. Lumen
has since acquired dedicated services for session management, provider
management, tracing, replay, operations and messaging.

Before changing Moderari's UI or adding further behaviour, its
responsibility should therefore be reconsidered in the context of the
current architecture.

The question is no longer:

> What does Moderari currently do?

The more useful question is:

> What should Moderari own now that the surrounding Lumen architecture
> exists?

------------------------------------------------------------------------

## Current Planning Direction

Moderari should remain concerned with the **construction and management
of the model-facing interaction context**.

It should not become the owner of:

-   provider lifecycle;
-   global model selection;
-   execution-session identity;
-   evidence interpretation;
-   operational control of the Stack.

Those responsibilities now belong elsewhere in Lumen.

A likely responsibility boundary is:

``` text
Session state + orchestration policy + available context
                         ↓
                      Moderari
                         ↓
             Model-facing interaction
```

Moderari should know enough about the active session to construct the
correct model interaction, but session ownership should remain outside
Moderari.

------------------------------------------------------------------------

# 4. Revisit Context Compaction

## Existing Position

Compaction originated as a practical mechanism for operating within
bounded model context.

That remains useful, but the wider Lumen architecture changes how
compaction should be considered.

A context reduction is not merely an internal optimisation. It changes
the information subsequently available to the model and may therefore
change later reasoning.

From a Reasoning Assurance perspective, that makes compaction an
observable transformation.

------------------------------------------------------------------------

## Planning Principle

Compaction should increasingly be treated as:

> **A deliberate transformation of session context whose inputs, outputs
> and consequences can be evidenced.**

Potential evidence includes:

-   context state before compaction;
-   reason compaction was initiated;
-   material retained;
-   material removed;
-   material transformed or summarised;
-   replacement continuity state;
-   token/context reduction;
-   compaction policy/version;
-   resulting model interaction;
-   subsequent observable changes in behaviour.

This does **not** imply storing every internal implementation detail
forever.

It means that Lumen should preserve enough evidence to answer questions
such as:

> What information did the model have before this response?

> Had Lumen compacted the context before the behaviour changed?

> What information survived that transformation?

> Could the compaction have materially affected the subsequent
> reasoning?

Those questions are directly relevant to Reasoning Assurance.

------------------------------------------------------------------------

## Open Moderari Questions

The Moderari review should resolve:

-   What state belongs to Moderari and what belongs to the Session?
-   Is compaction initiated by Moderari, requested by another service,
    or governed by policy?
-   What constitutes a compaction event?
-   What evidence should Vestigare receive about that event?
-   How is the replacement context associated with the originating
    session?
-   Should different models/providers permit different compaction
    policies?
-   Which parts of the existing Moderari UI remain operationally useful?
-   Which UI responsibilities now belong more naturally in Servire,
    Rogare or another service?

The objective should be a clear Moderari contract before substantial UI
work begins.

------------------------------------------------------------------------

# 5. Session → Provider → Model Ownership

## Current Limitation

The M0.1 model-selection architecture was intentionally sufficient for
the research distribution, but model selection remains effectively
associated with runtime-global provider state and reservation.

That becomes restrictive once multiple independent sessions are expected
to execute concurrently.

The next architecture should therefore move toward:

``` text
Session
  ↓
Provider Binding
  ↓
Model Binding
```

rather than:

``` text
Lumen Runtime
  ↓
Selected Model
```

------------------------------------------------------------------------

## Proposed Session Binding

A session should carry or resolve an execution binding conceptually
equivalent to:

``` text
session_id
provider_id
model_id
protocol / adapter
provider-specific execution metadata
```

The exact schema remains to be designed.

The important architectural decision is that the provider and model are
properties of an execution session rather than global properties of
Lumen.

------------------------------------------------------------------------

## Consequences

This permits independent sessions such as:

``` text
Session A
  Provider: Ollama
  Model: qwen2.5-coder:14b

Session B
  Provider: Ollama
  Model: gemma3:4b

Session C
  Provider: Colibri
  Model: qwen36-35b-a3b
```

without requiring one session's model selection to redefine the
execution environment of another.

This also creates a much clearer evidence model.

A recorded event can identify:

``` text
who/what initiated it
which Lumen session it belongs to
which provider handled it
which model executed it
which trace/event contains the evidence
```

------------------------------------------------------------------------

# 6. Provider Binding Must Not Mean "OpenAI API Everywhere"

Colibri demonstrates why provider abstraction should not be confused
with API homogenisation.

An OpenAI-compatible interface can be useful where a provider naturally
supports it, but Lumen should not assume that every provider will
expose:

-   identical transport;
-   identical model lifecycle;
-   identical context semantics;
-   identical streaming behaviour;
-   identical tool behaviour;
-   identical capability discovery;
-   identical failure semantics.

The abstraction should therefore be conceptual:

``` text
Session
  ↓
Provider Binding
  ↓
Provider Adapter / Protocol
  ↓
Model
```

rather than:

``` text
Everything
  ↓
OpenAI-compatible API
```

This allows Ollama, Colibri and future providers to retain their native
characteristics while presenting the execution capabilities Lumen
requires.

Provider-specific differences become explicit engineering concerns
instead of being hidden behind an abstraction that may eventually leak.

------------------------------------------------------------------------

# 7. Vestigare --- Concurrent Recording

## Current Architectural Pressure

A single active interaction naturally encourages concepts such as:

``` text
current session
current trace
selected model
active execution
```

Those assumptions do not survive a multi-session Lumen.

Vestigare must therefore become capable of receiving and recording
evidence from multiple independent execution streams concurrently.

------------------------------------------------------------------------

## Responsibility

Vestigare should remain an evidence recorder.

Its task is not to determine whether two events are logically related,
whether an agent behaved correctly, or whether a reasoning path was
justified.

Its task is to preserve sufficient evidence for those questions to be
answered later.

A simplified event identity might include:

``` text
event_id
timestamp
session_id
trace_id
parent_trace_id
correlation_id
source_service
provider_id
model_id
event_type
payload / evidence
```

This is illustrative rather than a final schema.

------------------------------------------------------------------------

# 8. Identity and Correlation

Concurrent execution makes identity a first-class engineering concern.

This is particularly important when Lumen later observes agentic or
swarm-style systems.

Different participants may carry different identifiers:

``` text
client_session_id
lumen_session_id
agent_id
execution_id
provider_request_id
trace_id
parent_trace_id
correlation_id
```

They do not necessarily need to share one universal identifier.

What matters is that the evidence preserves enough relationships to
reconstruct the activity.

Conceptually:

``` text
Agent A ── execution X ──┐
                         │
Agent B ── execution Y ──┼── Vestigare evidence
                         │
Agent C ── execution Z ──┘
                         ↓
                      Aestimare
                         ↓
             reconstructed relationship
```

This leads to an important principle:

> **Recording does not require complete understanding at recording
> time.**

Aestimare can later correlate identifiers, timestamps, parent/child
relationships and other evidence to reconstruct what happened.

This avoids making Vestigare responsible for interpreting systems it is
intended to observe.

------------------------------------------------------------------------

# 9. Why Concurrent Recording Matters for Reasoning Assurance

Traditional evaluation often concentrates on:

``` text
Prompt → Answer
```

Reasoning Assurance requires a richer record:

``` text
Request
  ↓
Session
  ↓
Context construction / transformation
  ↓
Provider and model selection
  ↓
Tool / agent / model activity
  ↓
Intermediate execution
  ↓
Response
```

For multi-agent systems, several of these paths may exist simultaneously
and may interact.

The correctness of the final answer does not by itself establish that
the execution path was acceptable.

A correct answer may have been produced through:

-   an unintended tool;
-   an unauthorised external resource;
-   incorrect intermediate reasoning;
-   a different model than expected;
-   an unexpected agent hand-off;
-   context contamination from another session;
-   an unsafe or prohibited action that happened to produce useful
    information.

Concurrent evidence capture is therefore not simply a scaling
requirement.

It is a prerequisite for reasoning assurance in systems where more than
one execution path can exist at the same time.

------------------------------------------------------------------------

# 10. Relationship Between the Three Workstreams

The three areas should be designed together even if they are implemented
sequentially.

``` text
                 SESSION
                    │
        ┌───────────┼───────────┐
        │           │           │
        ↓           ↓           ↓
    Moderari     Provider    Vestigare
        │         Binding       │
        │           │           │
        │           ↓           │
        │         Model         │
        │                       │
        └──── execution ────────┘
                    │
                    ↓
                 Evidence
```

Moderari needs session identity because it must manage the correct
context.

Provider/model selection needs session identity because execution
resources belong to the session.

Vestigare needs session identity because evidence from concurrent
executions must remain distinguishable.

The **session** therefore becomes the stable correlation boundary across
these concerns.

------------------------------------------------------------------------

# 11. Recommended Engineering Sequence

## Stage 1 --- Define Modern Moderari Responsibility

Document:

-   what Moderari owns;
-   what it consumes;
-   what it emits;
-   what state is session-scoped;
-   what it explicitly does not own.

Do this before UI restructuring.

------------------------------------------------------------------------

## Stage 2 --- Define Compaction Semantics

Specify:

-   trigger;
-   input state;
-   transformation;
-   output state;
-   evidence emitted;
-   relationship to the session;
-   relationship to later reasoning.

This should make compaction observable without unnecessarily coupling
Moderari to Vestigare internals.

------------------------------------------------------------------------

## Stage 3 --- Define Session → Provider → Model Contract

Design the authoritative ownership model for:

``` text
Session
Provider
Model
Protocol / adapter
Lifecycle
```

Determine which service owns each part of that state.

------------------------------------------------------------------------

## Stage 4 --- Implement Per-Session Model Selection

Demonstrate that two or more independent sessions can use different
models without interfering with each other's lifecycle or state.

Initially this can be proven using a single provider such as Ollama.

------------------------------------------------------------------------

## Stage 5 --- Make Vestigare Concurrent

Remove assumptions of one current trace/session.

Ensure events can be accepted, persisted and retrieved correctly when
multiple sessions are active simultaneously.

------------------------------------------------------------------------

## Stage 6 --- Exercise the Combined Architecture

Run concurrent sessions using different models.

Verify:

-   session isolation;
-   context isolation;
-   model binding;
-   lifecycle correctness;
-   trace separation;
-   correlation metadata;
-   terminal cleanup.

------------------------------------------------------------------------

## Stage 7 --- Introduce Multiple Provider Bindings

Once session/model ownership is stable, add provider diversity.

Initial useful combination:

``` text
Ollama
Colibri
```

The purpose is not simply to demonstrate two providers.

It is to validate that the provider-binding abstraction accommodates
genuinely different execution environments.

------------------------------------------------------------------------

## Stage 8 --- Broaden Client Support

Additional clients such as LibreChat can then be introduced against a
stable session architecture.

The client integration question becomes:

> How does this client establish, maintain and terminate a Lumen
> execution session?

rather than forcing client-specific behaviour into provider or model
management.

------------------------------------------------------------------------

# 12. Explicitly Deferred

The following should not drive the initial redesign:

-   adding many providers immediately;
-   supporting many client protocols immediately;
-   redesigning Aestimare before the evidence model exists;
-   forcing all providers through one transport protocol;
-   broad UI redesign before service responsibilities are settled;
-   altering the frozen M0.1 research reference to incorporate this
    work.

These capabilities should follow the architectural foundations rather
than determine them.

------------------------------------------------------------------------

# 13. Architectural Principles Emerging from the Planning

Several broader Lumen principles are reinforced by this work.

### Session identity is more fundamental than model identity

Models can change between sessions and providers. The session is the
stable execution boundary through which context, provider, model and
evidence can be correlated.

### Evidence capture and evidence interpretation should remain separate

Vestigare should preserve what happened. Aestimare should determine what
that evidence means.

### Context transformation is part of the reasoning path

Compaction can influence later behaviour and therefore belongs in the
evidence chain.

### Provider independence requires semantic abstraction, not forced protocol uniformity

Lumen should define what it needs from a provider without pretending
every provider behaves identically.

### Concurrency changes architecture, not merely throughput

Supporting multiple sessions means eliminating implicit global state and
making ownership explicit throughout the execution path.

------------------------------------------------------------------------

# 14. Working Definition of the Next Phase

The next phase of Lumen can be summarised as:

> **Move Lumen from managing an interaction to managing multiple
> independent, observable execution sessions.**

Each session should be able to possess its own:

-   context state;
-   provider binding;
-   model binding;
-   execution lifecycle;
-   trace/evidence stream.

Lumen should then be able to preserve enough evidence across those
independent streams for Aestimare to reconstruct and assess what
occurred.

This provides the architectural foundation for:

-   multiple providers;
-   multiple models;
-   multiple clients;
-   concurrent users or research experiments;
-   agent and swarm observation;
-   cross-session analysis;
-   stronger Reasoning Assurance.

------------------------------------------------------------------------

# 15. Immediate Next Design Task

The first implementation task should **not** yet be changing model
selection or Vestigare.

The next design task should be to write the modern **Moderari
responsibility and boundary contract**.

That exercise will establish:

``` text
what enters Moderari
what state Moderari owns
what is session-scoped
what Moderari changes
what evidence those changes produce
what leaves Moderari
```

Once that boundary is explicit, the Session → Provider → Model contract
can be designed against it, followed by concurrent Vestigare recording.

This keeps the next phase evidence-led and avoids allowing the
implementation of one feature to accidentally define the architecture of
the others.
