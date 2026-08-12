# Lumen Post-Rogare Development Roadmap

**Date:** 2026-08-11\
**Status:** Planned follow-up work after Rogare\
**Scope:** Servire, Repetere/Replay, and cross-component operational
logging

## 1. Purpose

This document records the development work to be undertaken **after
Lumen Rogare is complete**.

Rogare development itself is intentionally excluded from this roadmap.
Where a new cross-component convention is already known --- specifically
the service-owned **Clear Logs API** --- Rogare should implement that
convention during its initial development rather than requiring a later
retrofit.

The architectural invariant remains:

> **Each Lumen component should have one clearly bounded
> responsibility.**

Servire remains the operational control plane. Components own their own
behaviour, state, persistence and logs; Servire observes, correlates and
requests operations through component APIs rather than reaching into
component internals.

------------------------------------------------------------------------

## 2. Current Baseline

The following work is considered complete before Rogare development
begins:

-   the former Lumen orchestrator has become **Lumen Moderari**;
-   Lumen is the name of the complete reasoning-assurance ecosystem;
-   Moderari is a first-class module/service with its own repository;
-   Pontis, Vestigare, Repetere and Moderari have been exercised
    successfully end-to-end;
-   Servire configuration now launches Moderari from the new
    `Lumen-Moderari` codebase;
-   Servire top-level navigation follows the **Latin component name ---
    English responsibility** convention;
-   Moderari Operations and Checkpoints are capabilities within
    **Moderari --- Orchestrator**, rather than separate top-level
    components;
-   ordinary Pi traffic through the complete stack has been verified;
-   Replay divergence, transparent post-fork traffic, Pontis lazy Pi ACP
    attachment, tool execution and automatic Trace completion have
    previously been verified.

The immediate next development phase is Rogare. The items below begin
**after Rogare**.

------------------------------------------------------------------------

## 3. Post-Rogare Priority 1 --- Repetere Replay Integrity

### 3.1 Protect Source Trace Recordings

A **Prepared Replay** depends on the Trace recording from which it was
prepared. That dependency must be enforced as an integrity rule.

While a Trace recording is referenced by one or more Prepared Replays:

-   the recording must not be deletable from the Replay UI;
-   the delete control should be disabled or replaced by a clear
    protected-state indication;
-   the UI should identify that the recording is referenced by a
    Prepared Replay;
-   API-level deletion attempts must be rejected with a controlled
    conflict response;
-   protection must remain effective regardless of whether deletion is
    attempted through the UI or directly through an API;
-   once all dependent Prepared Replays are unstaged/deleted, the Trace
    recording becomes deletable again.

This prevents the invalid state currently demonstrated by a Prepared
Replay whose source recording had already been deleted, causing
`RecordingNotFoundError` during replay start.

### 3.2 Graceful Handling of Missing Source Recordings

Even with deletion protection, Repetere must handle historical or
externally-created invalid states safely.

If a Prepared Replay references a missing source recording:

-   do not raise an unhandled ASGI `500`;
-   mark or report the replay as **invalid / source recording missing**;
-   reject replay start with a controlled validation/conflict response;
-   present a useful message in the Replay UI;
-   log the condition as an operationally meaningful event.

### 3.3 Bound the Trace Recordings List

The **Trace Recordings** section of the Repetere UI must use a bounded, fixed-height display region rather than continuing to grow vertically as recordings accumulate.

As the number of Trace recordings increases:

- the Trace Recordings container should remain at a practical fixed/max height;
- overflow should be handled with an internal vertical scrollbar;
- the table header should remain usable while scrolling, preferably as a sticky header;
- the **Prepared Replays** section must remain readily reachable and must not progressively disappear further down the page simply because Trace history has grown;
- the behaviour should remain usable at normal desktop widths and degrade sensibly on smaller displays.

This is an operational usability requirement rather than a data-retention limit. Repetere may continue to expose the available Trace history; the UI should simply prevent that history from dominating the Replay workspace.

### 3.4 Standardise Replay Vocabulary

Remove the generic term **Experiment** from Replay-facing terminology
where it describes a Replay lifecycle object.

Preferred vocabulary:

  Concept                                   Standard term
  ----------------------------------------- ---------------------
  Captured behavioural source               **Trace Recording**
  Replay prepared/staged from a recording   **Prepared Replay**
  One execution of that replay              **Replay Run**
  Outcome of a run                          **Replay Result**
  First behavioural divergence              **Fork Point**

The aim is to make the UI, API, logs and documentation describe the
actual Replay lifecycle rather than a broader experimental concept.

------------------------------------------------------------------------

## 4. Post-Rogare Priority 2 --- Servire Runtime Health

Startup validation and runtime health must be treated as separate
concerns.

Startup validation answers:

> **Can the Lumen stack safely start?**

Runtime health answers:

> **Is the running Lumen stack currently capable of fulfilling its
> responsibilities?**

### 4.1 Continuous External Dependency Health

Servire should continuously evaluate required external dependencies
after the stack has started, including initially:

-   **Ollama / model provider**;
-   **MongoDB / persistence**.

A managed process being alive is not sufficient evidence that Lumen is
operational.

### 4.2 Stack Health States

Use clear aggregate semantics:

-   **HEALTHY** --- all required managed components are running and all
    required external dependencies are available;
-   **DEGRADED** --- managed components remain running, but one or more
    required dependencies are unavailable;
-   **STOPPED** --- the managed stack has been intentionally stopped;
-   **FAILED** --- a required managed component has failed or the
    required topology cannot be established.

Loss of Ollama or MongoDB while the stack is running should transition
the stack from **HEALTHY** to **DEGRADED**.

Servire should clearly identify the dependency responsible for
degradation.

### 4.3 Recovery Behaviour

Dependency loss must **not automatically stop the managed Lumen
services**.

The stack should remain running so that transient dependency failures
can recover without a destructive restart cycle. When the dependency
becomes available again, Servire should automatically return the
aggregate state to **HEALTHY**, assuming no other failure remains.

------------------------------------------------------------------------

## 5. Post-Rogare Priority 3 --- Service-Owned Log Management

Introduce a common **Clear Logs API** convention across the Lumen
component family.

### 5.1 Architectural Rule

Each component owns its own logs and logging lifecycle.

Servire must **not** directly truncate or delete another component's log
files. Instead, Servire requests the operation through that component's
API.

### 5.2 Components

The convention should apply to:

-   Moderari;
-   Repetere;
-   Vestigare;
-   Pontis;
-   Servire where appropriate;
-   Rogare from its initial implementation;
-   future Fiducia;
-   future Aestimare.

Rogare should therefore be built with the Clear Logs API from the
outset. Existing components can be retrofitted during this post-Rogare
phase.

### 5.3 API Behaviour

Standardise, as far as practical:

-   endpoint naming;
-   success response;
-   failure response;
-   operational logging of the clear request;
-   behaviour when logs are actively being exported/read;
-   treatment of multiple component-owned log files;
-   security/operational safeguards.

Servire should then expose the operation centrally, while preserving
component ownership.

------------------------------------------------------------------------

## 6. Post-Rogare Priority 4 --- Servire Operational Logging Improvements

### 6.1 Multiline Exception Capture

Servire's Operational Log must preserve complete multiline
exception/traceback information.

The recent Replay failure demonstrated that showing only:

`Exception in ASGI application`

without the associated traceback is insufficient for operational
diagnosis.

Servire should capture and display the traceback as part of the same
logical log event, while retaining sensible filtering and presentation.

### 6.2 Operational Log UI

Retain the revised dashboard ordering:

1.  Stack Health
2.  Stack Actions
3.  Operational Log
4.  Managed Services / External Dependencies
5.  Configuration

Ensure Operational Log controls remain stable as the log buffer fills.
In particular, **Pause / Export / Clear** should remain on the same
toolbar row at normal desktop widths rather than moving the Clear action
onto a second line.

### 6.3 Clear Logs Integration

Once service-owned Clear Logs APIs exist, Servire should provide a
coherent control surface for:

-   clearing an individual component's logs;
-   clearly identifying which component is being affected;
-   optionally clearing all component logs through coordinated API
    calls;
-   reporting partial failure rather than implying success if one
    component rejects or fails the request.

------------------------------------------------------------------------

## 7. Post-Rogare Priority 5 --- Servire Health and Event Correlation

The runtime-health work should become the practical foundation for the
previously identified cross-component health/event direction in Servire.

Servire should increasingly correlate rather than merely list state.
Examples include:

-   dependency lost / dependency restored;
-   service started / stopped / failed;
-   Replay started / completed / forked / invalid;
-   Trace recording started / completed;
-   checkpoint created / restored;
-   log clear requested / completed / failed.

This remains an operational responsibility: components publish or expose
their state; **Servire correlates and presents it**.

------------------------------------------------------------------------

## 8. Recommended Development Sequence After Rogare

1.  **Repetere integrity fix** --- protect Trace recordings referenced
    by Prepared Replays and handle missing sources gracefully.
2.  **Replay Trace-list containment** --- give the Trace Recordings section
    a fixed/max height with internal scrolling so Prepared Replays remain
    readily accessible as Trace history grows.
4.  **Replay vocabulary cleanup** --- replace Experiment terminology
    with Prepared Replay / Replay Run / Replay Result where appropriate.
3.  **Common Clear Logs API contract** --- define the small
    cross-component API convention, taking the Rogare implementation
    into account.
5.  **Retrofit Clear Logs APIs** --- Moderari, Repetere, Vestigare and
    Pontis; Servire itself where useful.
6.  **Servire Clear Logs controls** --- invoke component APIs rather
    than manipulating files.
7.  **Servire runtime dependency health** --- HEALTHY / DEGRADED /
    STOPPED / FAILED with Ollama and MongoDB runtime monitoring.
8.  **Servire multiline exception capture** --- preserve complete
    tracebacks in the operational log.
9.  **Servire health/event correlation** --- build on runtime health and
    operational events for a coherent stack-level view.
10.  **Full stack regression** --- Pi and Rogare traffic, tool use,
    Replay match/fork, Trace lifecycle, dependency loss/recovery and log
    management.

------------------------------------------------------------------------

## 9. Acceptance Criteria

This post-Rogare phase is complete when:

-   a Trace recording cannot be deleted while referenced by a Prepared
    Replay;
-   a missing Replay source produces a controlled user-visible state
    rather than HTTP 500;
-   the Trace Recordings section remains bounded and internally scrollable
    so Prepared Replays stay readily accessible as recording history grows;
-   Replay-facing terminology no longer relies on **Experiment** for
    core lifecycle objects;
-   Servire visibly enters **DEGRADED** if Ollama or MongoDB becomes
    unavailable after startup;
-   the stack remains running during transient external dependency loss
    and returns to **HEALTHY** on recovery;
-   each existing service exposes the agreed Clear Logs operation, with
    Rogare already conforming;
-   Servire clears component logs only through component-owned APIs;
-   Servire Operational Log preserves useful multiline exception
    tracebacks;
-   the complete stack remains operational through both Pi and Rogare
    after the changes.

------------------------------------------------------------------------

## 10. Architectural Principle

The work above should strengthen rather than blur component boundaries:

> **Components own capability. Servire owns operation and correlation.**

Repetere owns Replay integrity. Vestigare owns Trace recordings. Each
service owns its logs. Servire observes those capabilities, exposes
their operational state, and invokes their supported management APIs.

This keeps the Lumen ecosystem modular as the stack expands beyond
Rogare toward Fiducia and Aestimare.
