# Praebere N9.6 Runtime and Readiness Reconciliation Mini-Roadmap

**Status:** Complete for M0.1 — N9.6.1, N9.6.2 and N9.6.3 implemented and live-validated; N9.6.4 deferred to Future Development  
**Target:** Lumen External Research Distribution M0.1  
**Date:** 2026-09-02  
**Scope:** Praebere runtime persistence, provider reality checks, external Ollama boundary, model-residency ownership, idle model behaviour and provider-neutral readiness

---

## Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-09-02 | Nigel Catterall | Created N9.6 mini-roadmap with four staged milestones, MongoDB persistence, Pontis reconciliation and acceptance gates. |
| 0.2 | 2026-09-02 | Nigel Catterall | Added managed full-stack stop/restart verification, including MongoDB persisted-state checks and config-default restart behaviour. |
| 0.3 | 2026-09-02 | Nigel Catterall | Clarified that orphaned provider child processes are advisory degraded state for N9.6, logged/readiness-visible but not startup-blocking; Praebere UI display deferred to N9.7. |
| 0.4 | 2026-09-02 | Nigel Catterall | Recorded N9.6.2 implementation and corrected lifecycle cleanup so Praebere-only stop preserves state unless Pontis confirms zero active execution sessions. |
| 0.5 | 2026-09-04 | Nigel Catterall | Reconciled the mini-roadmap with validated N9.5 and later N9.6 lifecycle work; adopted external Ollama as a non-Lumen-managed dependency, retained first-ask demand residency and Praebere-owned model-residency release, bounded M0.1 closure to N9.6.1-N9.6.3, and moved the full provider-neutral readiness contract to nice-to-have/post-M0.1. |
| 0.6 | 2026-09-05 | Nigel Catterall | Closed N9.6.1-N9.6.3 following successful multi-client live validation; clarified Praebere ownership of runtime-global model reservation, selection and residency; retained Pontis authority for session identity/lifecycle; and moved N9.6.4 to Future Development. |

---

## 1. Purpose

N9.5 completed the runtime-global model-selection, reservation and active
execution-session lock path. Praebere is authoritative for model selection,
reservation, residency and the derived selection state. Pontis is authoritative
for session identity and lifecycle and supplies the live session facts against
which Praebere reconciles.

N9.6 reconciles that logical control-plane state with runtime reality. Praebere
must no longer rely only on startup configuration or in-memory state. It must be
able to persist its active state, restart safely, reconcile with Pontis, inspect
provider reality and report readiness truthfully.

The work was originally split into four staged milestones. Following the September 4
runtime reconciliation, **N9.6.1 through N9.6.3 form the M0.1 critical path**.
N9.6.4 remains a useful provider abstraction but is no longer required to close
M0.1; it is retained as a nice-to-have/post-M0.1 enhancement.

---

## 2. N9.6 Design Rule

Praebere must distinguish these sources of truth:

| Source | Authority |
|---|---|
| Praebere MongoDB runtime state | Authoritative selected model, model reservation, residency ownership and active execution registrations accepted by Praebere |
| Pontis session registry | Authoritative live Lumen session identity, lifecycle and execution-activity facts used for reconciliation |
| Provider reality | What the provider process and provider API actually report now |
| Configuration | Preferred/default startup intent, not authority over recovered runtime state |

Configuration may supply the preferred model and provider defaults. It must not
silently overwrite persisted selected model or active execution state after a
restart.

N9.6 must also distinguish managed full-stack lifecycle from Praebere-only
restart or crash recovery:

| Scenario | Expected Runtime Behaviour |
|---|---|
| Managed Lumen stack stop | Pontis closes all sessions; Praebere releases all reservations and execution locks, returns runtime state to a clean stopped state, and unloads only model residency it caused. Lumen never stops Ollama. |
| Managed Lumen stack restart | The clean stop completes first; on start, Praebere may return to `config.yaml` defaults/preferred model because no active execution sessions remain. |
| Praebere-only restart | Praebere must reload persisted runtime state and reconcile with Pontis before accepting normal model-selection changes. |
| Crash or unmanaged stop | Praebere must assume persisted active execution state may still be valid and must reconcile before unlocking. |

A managed stack stop is therefore an intentional runtime cleanup. A component
restart or crash is not.

---

## 3. Milestone N9.6.1 — MongoDB Runtime-State Persistence

### Goal

Persist Praebere runtime state in MongoDB whenever authoritative state changes.
This makes selected model and active execution-session state durable across
Praebere stop, crash and restart.

### Required State

Persist at least:

| Field | Requirement |
|---|---|
| Provider identity | Current provider type/name, such as Ollama |
| Provider endpoint | Endpoint Praebere is using |
| Preferred/configured model | Startup/default model from configuration |
| Selected model | Runtime-global authoritative selected model |
| Reserved sessions | Open Pontis session IDs for which Praebere has accepted the selected runtime-global model |
| Active execution sessions | Session IDs registered as active execution sessions |
| Selection state | `available`, `reserved`, `locked`, or `reconciliation_pending`, derived from reservation, active-execution and reconciliation state |
| Model residency ownership | Whether selected-model residency was caused by Praebere, pre-existed externally, or is not resident/unknown |
| Generation/version | Monotonic state version for reconciliation/audit |
| Updated timestamp | Last committed state change |

`selection_locked` may be stored as evidence, but it must be derived at runtime:

```text
selection_locked = active_execution_session_ids is not empty
```

`reserved` is distinct from `locked`: an open session may reserve the selected
model before its first ask, while `locked` means at least one session is actively
using the model for execution.

### Acceptance Gate

- Selecting a model persists the selected model.
- Activating an execution session persists the active session ID.
- Releasing an execution session persists removal of that session ID.
- Restarting Praebere restores selected model and active execution sessions from
  MongoDB.
- Restarting Praebere must not silently unlock when persisted active execution
  sessions exist.
- A managed full-stack stop persists an intentional clean runtime state with no
  active execution sessions and no derived selection lock.
- After a managed full-stack restart, Praebere starts from `config.yaml`
  defaults/preferred model rather than restoring a stale active execution lock.
- The managed full-stack stop behaviour is verified by inspecting Praebere's
  persisted MongoDB state after stack stop and before the next start.
- Corrupt or unreadable persisted state produces an explicit degraded/recovery
  state, not an apparently clean runtime.

---

## 4. Milestone N9.6.2 — Pontis Reconciliation on Start/Restart

### Goal

On Praebere start or restart, reconcile persisted runtime state against Pontis's
authoritative session registry before normal model-selection changes are
accepted.

### Required Behaviour

Praebere startup must:

1. load persisted MongoDB runtime state;
2. enter `reconciliation_pending` if persisted active execution sessions exist;
3. query Pontis for authoritative active execution sessions;
4. remove persisted execution registrations that Pontis confirms are no longer
   active;
5. restore any Pontis-active execution registration missing from Praebere's
   persisted state;
6. recalculate the active execution count and derived lock;
7. persist the reconciled result; and
8. expose reconciliation state through readiness/control responses.

If Pontis is unavailable and persisted active execution sessions exist, Praebere
must remain fail-safe and locked. It must not assume the sessions disappeared.

### Acceptance Gate

- Praebere restart with one Pontis-active execution session remains locked.
- Praebere restart with multiple Pontis-active execution sessions restores the
  correct count.
- Praebere restart with only established, non-executing Pontis sessions remains
  unlocked.
- A stale persisted registration is removed only after Pontis confirms it is no
  longer active.
- A Pontis-active registration missing from Praebere's persisted state is
  restored.
- Model selection is rejected while reconciliation is pending.
- Failed reconciliation is observable and retryable.
- A Praebere-only stop with Pontis-active execution sessions preserves selected model,
  active execution registrations, selection lock and provider state.
- Clean stopped state is persisted only after Pontis confirms zero active execution
  sessions.

---

## 5. Milestone N9.6.3 — External Ollama and Runtime-Reality Checks

### Goal

Reconcile Praebere model-selection and residency state against the externally operated
Ollama endpoint without making Lumen responsible for the Ollama process lifecycle.

### Superseding M0.1 Boundary

The later Pontis/Praebere lifecycle decision supersedes the earlier
Praebere-started-versus-external Ollama ownership model:

> **Ollama is external infrastructure. Lumen verifies the configured Ollama endpoint
> but does not start or stop the Ollama process.**

Praebere still owns a narrower and important resource boundary: **model residency
caused by Praebere**. If Praebere demand-loads the selected model on first ask, it may
unload that model after the final active execution session ends. If the model was
already resident before Lumen used it, Praebere must leave that residency alone.

### Required Behaviour

- Praebere verifies the configured Ollama endpoint during lifecycle startup.
- Endpoint unavailability is an explicit lifecycle-start failure.
- Servire rolls back only services started by that stack-start attempt.
- Lumen does not start Ollama as a recovery side effect.
- Lumen does not stop Ollama during stack stop or Praebere stop.
- Model selection records authoritative intent but does not force residency.
- On first ask, Praebere checks whether the selected model is resident.
- Praebere loads the selected model only when it is not already resident.
- Praebere records model-residency ownership only when Praebere issued that load.
- Concurrent execution sessions using the same reserved model share residency.
- On final active-execution release, Praebere unloads only residency it caused.
- A failed owned-model unload is degraded/partial completion; it does not authorise
  stopping Ollama.
- Provider/model process-reality failures remain explicit and observable.

### Acceptance Gate

- [x] External Ollama availability is checked during Praebere/Lumen startup.
- [x] An unavailable Ollama endpoint fails the applicable startup and triggers bounded
  Servire rollback without attempting to start Ollama.
- [x] Normal stack/Praebere stop does not stop the external Ollama process.
- [x] Selecting a model does not itself load the model.
- [x] The first ask demand-loads a selected non-resident model.
- [x] A model already resident before Lumen execution is not marked as Praebere-owned.
- [x] Multiple active sessions share the same authoritative model residency correctly.
- [x] Final execution release unloads the model only when Praebere caused residency.
- [x] Failed owned-model unload is reported as degraded/partial completion rather than
  escalating into provider-process termination.
- [x] The known N9.6.3 defects identified during September 4 live testing are corrected
  and the full lifecycle validation pass is rerun cleanly.

**Current state:** complete for M0.1. Live validation confirmed model reservation,
multi-client conflict handling, first-ask demand loading, successful execution and
final-session release/unload without a 5xx lifecycle failure.

---

## 6. Milestone N9.6.4 — Provider-Neutral Readiness Contract

**Status: NICE-TO-HAVE / POST-M0.1. NOT A RELEASE BLOCKER.**

### Goal

Retain a future provider-neutral readiness capability that can report whether Praebere
can safely execute with the selected runtime model without coupling callers to Ollama.

### Required Readiness Distinctions

Praebere must distinguish:

| State | Meaning |
|---|---|
| Selected | Lumen's runtime-global selected model |
| Available | Provider has the model installed/listed |
| Resident/loaded | Provider currently has the model loaded in memory |
| Warmable | Provider can load the selected model on demand |
| Ready | Provider can accept execution for the selected model |
| Degraded | Provider is reachable but state is inconsistent or incomplete |
| Unavailable | Selected model cannot currently be executed |

Idle or non-resident model behaviour must not be treated as a lost selection.
An Ollama model may unload after inactivity while remaining selected and
available. Readiness should report that as `warmable` or equivalent, not as an
unselected runtime.

### Proposed Control Response

Praebere should expose readiness through a control-plane command and API
equivalent to:

```text
\obt praebere readiness
```

The response should include at least:

```json
{
  "provider": "ollama",
  "provider_reachable": true,
  "provider_lifecycle": "external_infrastructure",
  "model_residency_ownership": "praebere_loaded",
  "selected_model": "qwen2.5-coder:14b-32k",
  "preferred_model": "qwen2.5-coder:14b-32k",
  "model_available": true,
  "model_resident": false,
  "ready_for_execution": true,
  "readiness_state": "warmable",
  "selection_locked": false,
  "active_execution_sessions": 0,
  "reconciliation_state": "complete",
  "warnings": []
}
```

### Acceptance Gate

- Readiness reports provider reachability.
- Readiness reports selected model and preferred model distinctly.
- Readiness confirms whether the selected model appears in provider reality.
- Readiness distinguishes available from resident/loaded.
- Readiness reports idle/non-resident selected models as warmable when the
  provider can load them normally.
- Readiness reports persisted/Pontis reconciliation state.
- Readiness reports the external provider-lifecycle boundary and, separately, model-residency ownership.
- Readiness remains provider-neutral even when the first implementation is
  Ollama-specific.

---

## 7. Development Sequence

| Order | Milestone | Component Focus | Progression Rule |
|---|---|---|---|
| 1 | N9.6.1 MongoDB persistence | Praebere | **Complete.** Durable runtime state is part of the M0.1 lifecycle. |
| 2 | N9.6.2 Pontis reconciliation | Praebere, Pontis, Nuntius | **Complete and live-validated.** |
| 3 | N9.6.3 External Ollama/runtime reality | Praebere, Servire, Pontis | **Complete and live-validated.** |
| 4 | N9.6.4 Provider-neutral readiness | Praebere, display consumers | **Nice-to-have/post-M0.1.** Not required to close the research distribution. |

Each stage should produce:

- code changes;
- focused tests;
- one live validation pass;
- updated version table entries in affected documents;
- a clean package before the next stage starts.

---

## 8. Known Questions

| Question | Current Position |
|---|---|
| Should Praebere automatically kill orphaned `llama-server` processes? | Not by default for M0.1. Prefer visible degraded state plus explicit administrative cleanup. |
| Should selected model be cleared if provider no longer lists it? | No. Preserve selected model, report unavailable, and require explicit operator/model-selection action. |
| Should configuration override persisted selected model on restart? | No. Configuration is preference/default, persisted runtime state is recovery evidence. |
| What if MongoDB is unavailable at Praebere startup? | Praebere should enter degraded/recovery state and reject unsafe model-selection changes until persistence/reconciliation is available. |
| What if Pontis is unavailable during reconciliation? | If persisted active execution sessions exist, Praebere remains locked and reconciliation pending. |
| Should readiness trigger model loading? | Prefer no for the readiness query itself. Report warmable versus resident; loading should be explicit or occur on execution. |
| Should Pontis admit new external work after stack shutdown has begun? | Future Development/UI polish: add an explicit shutdown-quiescing state so late requests are rejected clearly (for example, HTTP 503) rather than creating a session during teardown. |

---

## 9. Completion Boundary

For **M0.1**, N9.6 is complete when:

- Praebere persists runtime state to MongoDB on every selected-model and active
  execution-session state change;
- Praebere restart validates persisted state before accepting normal
  model-selection changes;
- Praebere reconciles active execution-session state with Pontis authority;
- persisted stale locks cannot remain indefinitely after Pontis confirms no
  active execution sessions exist;
- active Pontis execution sessions cannot be silently unlocked by restarting
  Praebere;
- managed full-stack stop/restart intentionally closes sessions, releases
  Praebere locks and persists the clean state;
- Praebere distinguishes configured/preferred model from selected runtime model;
- Praebere distinguishes selected model from loaded/resident provider state;
- the configured Ollama endpoint is treated as external infrastructure and
  Lumen never starts or stops the Ollama process;
- first ask demand-loads a selected non-resident model;
- final execution release unloads only model residency caused by Praebere;
- the known N9.6.3 defects are fixed and final live lifecycle validation passes.

The compact Praebere operational UI consumes these concrete states but is the
subsequent N9.7 work item; it is not part of the N9.6 completion boundary.

The full provider-neutral `available/warmable/ready/degraded/unavailable` readiness
taxonomy is explicitly **outside the M0.1 completion boundary** and remains a
post-M0.1 enhancement.


---

## 10. M0.1 Closeout Decision — 2026-09-05

For M0.1, this mini-roadmap is considered closed when:

- [x] N9.6.1 persistence behaviour is complete.
- [x] N9.6.2 receives its final live reconciliation acceptance evidence.
- [x] N9.6.3 known defects are corrected and the final runtime lifecycle validation
  passes.
- [x] The external-Ollama boundary is used consistently: Lumen checks availability but
  does not start or stop Ollama.
- [x] Model selection and model residency remain distinct; first ask performs demand
  loading and Praebere releases only residency it caused.
- [x] N9.6.4 is explicitly recorded as nice-to-have/post-M0.1 rather than silently
  abandoned.

Completion of the full N9.6.4 provider-neutral readiness taxonomy is **not** required
to close N9.6 for the External Research Distribution M0.1.
