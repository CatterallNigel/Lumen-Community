# Per-Session Model Ownership and Concurrent Trace Recording

**Status:** Future development  
**Date recorded:** 4 September 2026  
**Target release:** To be determined  
**M0.1 impact:** Not an M0.1 blocker

## Purpose

This document records two related future architectural changes:

1. allowing each Pontis execution session to select and own its model independently; and
2. allowing Vestigare to record multiple active execution sessions concurrently.

The changes are related because a concurrent trace must preserve the model and execution context belonging to its own session. Per-session model ownership should therefore be implemented before concurrent Vestigare recording.

## Current Behaviour

Lumen currently uses a runtime-global provider/model selection:

- Pontis owns execution-session identity;
- Praebere owns the authoritative selected-model state and selection lock;
- the first selected model is reserved until all active execution sessions have ended;
- concurrent sessions inherit the same locked model;
- Praebere loads the selected model on the first real ask rather than merely when a session is opened; and
- Vestigare permits only one globally active recording, bound to one explicit session/owner.

This behaviour is appropriate for M0.1 and avoids prematurely introducing model-resource arbitration into the external research distribution.

## Change 1: Per-Session Model Ownership

### Objective

Each active Pontis session must be able to select and retain its own model independently of other sessions. Requests belonging to one session must never acquire another session's model implicitly.

### Required Architectural Changes

#### Pontis

- Store provider and model identity as part of authoritative session state.
- Resolve the session's model for every request.
- Preserve lazy execution initialisation: opening a session must not create ACP, select or load a model, or send `READY`.
- Create and configure the execution path on the first real ask.
- Reject attempts to change a session's model after execution has begun unless an explicit model-change lifecycle is subsequently designed.
- Expose the selected model through session inspection and operational commands.

#### Praebere

- Replace the single global selection lock with session-to-model usage state.
- Track which active sessions reference each model.
- Introduce reference-counted or equivalent model ownership so a model is not unloaded while an active session still depends upon it.
- Reconcile model residency independently from logical session ownership.
- Retain provider lifecycle and discovery authority.
- Continue to observe the existing ownership constraint: Praebere must not stop an externally started provider.

#### Moderari

- Accept the provider/model resolved for the individual request session.
- Avoid introducing provider discovery or lifecycle responsibilities into Moderari.
- Prevent global configuration from silently overriding the session model.

#### Rogare and Pi

- Display the model belonging to the current session.
- Allow model selection before that session's first real ask.
- Make model lock state session-specific rather than runtime-global.
- Show resource or availability conflicts explicitly.

#### Vestigare

- Record the provider and model identity belonging to the traced session.
- Preserve this identity as execution provenance rather than deriving it later from global runtime state.

#### Repetere

- Identify the original session model from recorded provenance.
- Distinguish replay using the original model from replay using an explicitly substituted model.
- Record model substitution as part of replay provenance.

#### Fiducia

- Bind scheduled execution and replay work to an explicit provider/model policy.
- Support coexistence between scheduled work and interactive sessions without silently changing either session's model.

#### Servire

- Reconcile multiple session/model relationships during shutdown, restart and rollback.
- End sessions and release model references in a deterministic order.
- Report residual provider processes or model residency as degraded state without corrupting logical ownership state.

### Resource-Arbitration Requirement

Independent model selection does not necessarily mean that every selected model can be resident simultaneously. The implementation must distinguish:

- **logical ownership:** the provider/model selected for a session;
- **execution availability:** whether the model can currently accept work; and
- **physical residency:** whether the model is loaded into local memory or accelerator resources.

On constrained local hardware, two sessions may legitimately own different models even when only one model can be resident at a time. The arbitration policy must be explicit: queue, reject, swap models, or use another provider. Silent eviction of a model required by an active session is not acceptable.

### Estimated Effort

| Work | Estimate |
| --- | ---: |
| Core implementation | 3–5 working days |
| Cross-service integration and lifecycle testing | 2–3 working days |
| **Realistic total** | **5–8 working days** |

A narrow implementation that permits only one locally resident model at a time could potentially be completed in 3–4 working days. It would not, however, provide genuinely independent concurrent model execution and could serialize or interrupt sessions using different models.

## Change 2: Concurrent Vestigare Recording

### Objective

Vestigare must be able to record several active execution sessions simultaneously, while keeping their events, metadata, lifecycle and failures isolated.

### Proposed State Model

Vestigare should replace its single global active-recorder state with an explicit mapping similar to:

```text
session_id -> recording_id -> recorder state
```

The exact persistence representation remains an implementation decision, but session identity must be the primary routing boundary.

### Required Changes

- Maintain independent recorder state for every actively traced session.
- Route Nuntius trace events using explicit session and recording correlation.
- Isolate buffers, output streams, file handles and locks between recordings.
- Allow one recording to stop or fail without affecting other recordings.
- Recover and reconcile incomplete recorder state after restart.
- Require an explicit target when several eligible sessions exist.
- Reject Trace start when no eligible session exists.
- Correct initial Trace-status synchronisation, including the currently known `404` condition.
- Expose all active recordings and their session/model associations in Rogare.
- Allow Repetere to select the correct completed recording independently.
- Define ownership and permission behaviour for multiple clients or users before multi-user deployment.

### Estimated Effort

| Work | Estimate |
| --- | ---: |
| Core implementation | 2–3 working days |
| Integration, concurrency and failure testing | 1–2 working days |
| **Realistic total** | **3–5 working days** |

## Recommended Implementation Order

1. Define and implement per-session provider/model state in Pontis.
2. Introduce session-aware model ownership and resource arbitration in Praebere.
3. Propagate the session model through Moderari and record it in Vestigare provenance.
4. Update Rogare and Pi session presentation and selection behaviour.
5. Update Repetere and Fiducia contracts.
6. Implement Vestigare's concurrent recorder registry and lifecycle.
7. Complete integrated multi-session lifecycle and recovery testing.

Implementing per-session model ownership first prevents Vestigare's recording contract from being redesigned twice.

## Combined Planning Estimate

| Scope | Estimate |
| --- | ---: |
| Per-session model ownership | 5–8 working days |
| Concurrent Vestigare recording | 3–5 working days |
| **Combined engineering estimate** | **8–12 working days** |

For planning purposes, reserve **two working weeks plus approximately three days of contingency**, or a **two-to-three-week calendar window**. The contingency is primarily for cross-service lifecycle behaviour rather than the basic data-model changes.

## Required Verification Scenarios

The work should not be considered complete until at least the following scenarios pass:

1. Two sessions select and use the same model concurrently.
2. Two sessions select different models and execute according to the defined arbitration policy.
3. Both sessions are traced concurrently and produce isolated recordings.
4. Ending one session releases only its model reference and recorder.
5. A failed request or recorder does not corrupt the other active session.
6. Replay identifies the original session model correctly.
7. An explicit replay-model substitution is recorded in provenance.
8. Stack shutdown ends sessions, stops recordings and releases owned model resources deterministically.
9. Startup reconciliation identifies incomplete sessions, recordings and model residency without inventing ownership.
10. An externally started provider is not stopped by Praebere or Servire.

## Principal Risks

- Local GPU or system memory may be insufficient for several resident models.
- Provider behaviour may differ when several models or requests are active.
- Global state may remain implicitly embedded in existing service code or UI assumptions.
- Shutdown and failure recovery may expose reference-count or ordering defects.
- Replay comparability may be weakened if model substitution is not made explicit.
- Trace events could be attributed to the wrong session unless correlation is mandatory at every boundary.

These risks are manageable, but they justify treating the change as a cross-service architectural increment rather than two isolated feature modifications.
