# Lumen Current Status

**Status:** Current\
**Purpose:** Concise maintained snapshot of Lumen's present engineering
state\
**Snapshot date:** 2026-08-21

------------------------------------------------------------------------

## What Lumen Is Today

Lumen is an emerging **Reasoning Assurance Service** composed of bounded
services around probabilistic model execution.

The principal execution and operational stack is working end-to-end.
Current development emphasis is moving from establishing
execution/reproduction infrastructure toward **behavioural assessment
through Aestimare**.

For architectural authority, use:

-   `docs/architecture/core/ARCHITECTURE.md`
-   `docs/architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md`
-   `docs/architecture/ecosystem/LUMEN_SERVICE_MATRIX.md`

This page is deliberately short and time-sensitive.

------------------------------------------------------------------------

## Service Status

  ----------------------------------------------------------------------------
  Service                 Present status          Current role / direction
  ----------------------- ----------------------- ----------------------------
  **Moderari**            Implemented / active    Model-request orchestration
                                                  and controlled intervention

  **Pontis**              Implemented / active    Client/session bridge and
                                                  protocol/capability path

  **Praebere**            Implemented / active    External model-provider
                                                  abstraction, readiness and
                                                  lifecycle

  **Vestigare**           Implemented / active    Trace and execution evidence
                                                  capture

  **Repetere**            Implemented / active    Controlled Replay and
                                                  divergence evidence

  **Fiducia**             Implemented / early     Coordinates repeated Replay
                          capability              execution; integrated with
                                                  Servire

  **Aestimare**           Active                  Behavioural assessment of
                          design/development      accumulated evidence

  **Rogare**              Implemented / active    Human-facing
                                                  conversational/operational
                                                  client

  **Servire**             Implemented / active    Operational control plane
                                                  and stack lifecycle

  **Audire**              Proposed architecture   Distributed operational
                                                  transport for future
                                                  multi-host operation
  ----------------------------------------------------------------------------

Refer to the service matrix for current ports and dependencies.

------------------------------------------------------------------------

## Operational State

The Lumen stack has reached end-to-end operation across the principal
implemented services.

Current operational architecture includes:

-   model-provider readiness/lifecycle through Praebere;
-   orchestration through Moderari;
-   bridging/session handling through Pontis;
-   execution evidence through Vestigare;
-   Replay through Repetere;
-   repeated Replay coordination through Fiducia;
-   user interaction through Rogare;
-   stack lifecycle and operational control through Servire.

This establishes the infrastructure needed to begin accumulating
controlled behavioural evidence for Aestimare.

------------------------------------------------------------------------

## Current Development Focus --- Aestimare

Aestimare is the major active architectural/development focus.

Its governing principles currently include:

-   **Assess behaviour, not intelligence.**
-   Treat a single execution as an observation rather than a behavioural
    conclusion.
-   Use repeated controlled executions to form experiments.
-   Characterise behaviour from multiple experiments/evidence sets.
-   Use longitudinal evidence to detect behavioural change.
-   Assess sufficiency where a task permits multiple acceptable
    outcomes.
-   Preserve execution context/provenance so behavioural change can be
    distinguished from changed conditions.
-   Allow specialist internal assessment engines to mature independently
    rather than force all assessment into one monolithic algorithm.

The proposed internal specialist-engine terminology is **Peritus**
(singular), **Periti** (plural), collectively **Periti Assessores**.

This is an Aestimare internal design direction, not an expansion of the
top-level Lumen service family.

------------------------------------------------------------------------

## Evidence Generation

The immediate prerequisite for useful Aestimare work is a controlled
corpus of executions.

The current approach begins with tasks having definitive answers, then
progresses toward prompts where:

-   multiple outcomes may be sufficient;
-   tool use may vary;
-   reasoning paths may diverge;
-   context can materially affect behaviour.

Fiducia provides the mechanism for repeated controlled Replay activity;
Vestigare and Repetere provide the underlying observation/reproduction
evidence.

------------------------------------------------------------------------

## Architectural Direction --- Audire

All Lumen services communicate over HTTP and need not permanently reside
on one host.

Servire currently operates services in a topology where local process
output can be directly available. Audire is the proposed architectural
answer for distributed operation.

The intended direction is a small operational transport layer capable of
carrying:

-   stdout/stderr-style operational messages;
-   lifecycle commands;
-   acknowledgements;
-   service state/control messages.

Audire is not currently part of the implemented critical path and should
be treated as **Proposed**, not as current runtime behaviour.

------------------------------------------------------------------------

## External Capability Integration

The architecture supports external tool/capability providers without
making those providers part of Lumen.

The governing rule is:

> External providers return capability results or semantics, not Lumen
> prompts.

Moderari remains responsible for Lumen-controlled
orchestration/instruction behaviour, while Pontis provides the
integration path.

The MEG semantic-provider work is an example of this architectural
pattern rather than a special-case dependency.

------------------------------------------------------------------------

## Current Documentation State

The Community repository has completed a structural and high-priority
documentation housekeeping pass.

Completed work includes:

-   reorganised service documentation;
-   service-level README indexes;
-   canonical current ecosystem topology;
-   canonical service matrix;
-   documentation status/lifecycle convention;
-   superseded-document register;
-   new canonical Lumen architecture;
-   classification of older architecture, UI Vision and Lumen Console
    documents as historical where appropriate.

The original architecture plus its two appendices are retained under
`docs/architecture/core/history/`.

------------------------------------------------------------------------

## Deferred

### Contribution / Development Entry Point

A formal `CONTRIBUTING.md` and external developer setup guide are
intentionally deferred until Lumen is ready to invite external code
contributions.

Public visibility of the Community repository does not by itself mean
the engineering contribution contract is ready.

------------------------------------------------------------------------

## Maintaining This Page

This document is a **snapshot**, not a historical ledger.

When status materially changes:

1.  update this page;
2.  update the service matrix where maturity/dependencies change;
3.  update the relevant service documentation;
4.  preserve meaningful historical milestones in service version history
    or the Engineering Diary.

Do not turn this page into another Engineering Diary or roadmap.

------------------------------------------------------------------------

## Current Summary

> Lumen's execution, evidence-capture, Replay, repeated-execution and
> operational foundations are in place. The next major step is to turn
> that accumulated evidence into behavioural assessment through
> Aestimare.
