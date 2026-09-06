# Pontis Session Management and Praebere Model-Lock Release

**Status:** Closed — N9.5/N9.6.2/N9.6.3 implemented and live-validated  
**Target:** Lumen External Research Distribution M0.1 — N9.6.3 lifecycle completion  
**Date:** 2026-09-04  
**Scope:** Pontis session authority, atomic model reservation, explicit termination, deferred-release reconciliation, Praebere demand residency and external Ollama boundary

---

## Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-08-31 | Nigel Catterall | Recorded the N9.5 session-lifecycle defect, agreed Rogare lifecycle, proposed external-client heartbeat, known transport uncertainty, required investigation and acceptance criteria. |
| 1.1 | 2026-08-31 | Nigel Catterall | Added Praebere active-state persistence, restart reconciliation, fail-safe recovery state, execution-state reset commands and expanded restart/reset acceptance requirements. |
| 1.2 | 2026-09-01 | Nigel Catterall | Replaced the proposed client-heartbeat design with the agreed Pontis-owned session-management UI and control commands; defined self-service closure, operator orphan cleanup, authorisation, audit, Praebere release and revised acceptance criteria. |
| 1.3 | 2026-09-02 | Nigel Catterall | Added per-session Provider tools control from Rogare/Pontis, model/tool compatibility handling, fatal provider tool-rejection session cleanup and final N9.5 completion evidence. |
| 1.4 | 2026-09-04 | Nigel Catterall | Added N9.6.2 deferred-release reconciliation and N9.6.3 atomic cross-client model reservation, external-Ollama availability gating, first-ask demand loading and Praebere residency ownership. |
| 1.5 | 2026-09-04 | Nigel Catterall | Reconciled M0.1 closeout status: N9.6.2 requires final acceptance evidence; N9.6.3 requires the known September 4 bug fixes plus final live lifecycle validation; the full provider-neutral N9.6.4 readiness taxonomy is explicitly nice-to-have/post-M0.1 and does not block this lifecycle boundary. |

---

## 1. Purpose

N9.5 established runtime-global model selection and locking through Pontis,
Nuntius and Praebere. Live validation demonstrated that the first model
interaction activates an execution session and that a different model cannot be
selected while an active execution session exists.

The same validation exposed an incomplete session-termination boundary. After
the standalone Pi client was closed, Praebere continued to report the selected
model as locked. Pontis had activated the Pi execution session but received no
explicit termination operation and therefore had not released the Praebere
execution registration.

This document defines the agreed M0.1 lifecycle. Pontis exposes explicit session
termination to clients and operators rather than attempting to infer client
liveness through a callback heartbeat. This gives the component that owns
session authority the corresponding ability to inspect, end and recover its
sessions.

Final N9.5 validation also introduced one further session-scoped compatibility
requirement: a client session must be able to choose whether Pontis injects ACP
tools into provider requests. Some provider/model combinations are valid for
ordinary text execution but reject OpenAI-style tool declarations. N9.5 therefore
requires explicit per-session Provider tools control rather than treating tool
injection as unconditional.

---

## 2. Authoritative Session and Lock Rules

Pontis is the authority for Lumen session identity and session lifecycle.
Praebere is the authority for runtime-global provider/model state and the active
execution-session count.

Pontis atomically owns the runtime-global model reservation. Praebere owns model
execution activation and any model residency it caused. A reservation may exist before
the first ask; this prevents two clients from believing they selected different models.

The required distinction is:

```text
Established session
    Pontis session_id exists
    May have no reservation or reserve the authoritative runtime-global model
    Does not by itself create Praebere execution activation or model residency

Active execution session
    First model interaction has occurred
    Pontis has activated the session through Nuntius/Praebere
    Contributes to the Praebere model lock

Closed session
    Pontis has ended the session
    Pontis has released its Praebere execution registration
    Does not retain a model reservation
```

When the final active execution session is released, Praebere releases its execution
lock and unloads the model only when Praebere caused that residency. Pontis releases
the model reservation only when the final session reserving that model closes. Ending
one session must not free the reservation or residency while another relevant session
remains.

---

## 3. N9.5 Evidence and Resolved Defects

Live N9.5 testing demonstrated:

- an external HTTP/Pi session selected Model A;
- Pontis activated that session on its first model interaction;
- Praebere locked the runtime-global selection to Model A;
- a concurrent Rogare session observed the same authoritative model and lock;
- an attempted change to Model B returned `409 Conflict`;
- Model A remained selected and the active execution-session count remained
  correct for the session known to Praebere.

Initial testing showed that after Pi was closed, a subsequent
`\obt praebere models` request from Rogare still reported the model as locked.

The defect was therefore:

> Pontis activates an external client's execution session but does not yet have
> explicit client and operator controls to end or recover that session and
> release its Praebere execution registration.

The N9.5 model-selection and lock-acquisition behaviour is proven. Subsequent
implementation added Pontis-owned session inspection, explicit session end,
operator force-close, Rogare End Session, Rogare New Session lifecycle cleanup
and Praebere release on active execution-session closure.

Final validation also proved the Provider tools compatibility path:

- a Rogare session using `gemma3:4b` with Provider tools enabled received the
  expected provider rejection because that provider/model combination does not
  support tool declarations;
- Rogare ended the affected session and reported that a new session should be
  started with Provider tools disabled; and
- a new Rogare session with Provider tools disabled successfully used
  `gemma3:4b` through the same Pontis/Praebere model-selection path.

---

## 4. Agreed Rogare Lifecycle

Rogare uses explicit session lifecycle operations for M0.1.

The agreed lifecycle is:

- add an **End Session** button to Rogare;
- **End Session** requests authoritative session closure from Pontis;
- starting **New Session** ends the current session before establishing the
  replacement session;
- Rogare service stop ends its current session as part of normal shutdown;
- Rogare service restart performs the normal stop/end-session lifecycle before
  establishing any new session;
- Pontis releases the Praebere execution registration when the Rogare session
  ends;
- closing one Rogare session must not close or release any unrelated external,
  Replay or other Rogare session.

The session-end operation must remain outside conversational model execution and
Vestigare Trace turns.

### 4.1 Rogare Failure Boundary

The agreed explicit lifecycle covers normal End Session, New Session, stop and
restart behaviour. If Rogare or its host process terminates without executing
normal shutdown, the Pontis UI and administrative commands provide the M0.1
recovery path for inspecting and closing the resulting orphaned session.

---

## 5. Agreed M0.1 Session-Management Solution

Pontis must provide explicit session inspection and termination. M0.1 will not
depend on Pontis posting a heartbeat to an external client.

The supported lifecycle is:

1. Pontis creates and owns the authoritative `session_id`.
2. The first successful model selection atomically reserves that model in Pontis;
   selecting it does not load it into Ollama.
3. A client may explicitly end its own session through a Pontis control command.
4. Rogare ends its session through its UI lifecycle.
5. An authorised operator may inspect all Pontis sessions and close a confirmed
   abandoned or orphaned session through the Pontis UI or administrative command.
6. Pontis releases the corresponding Praebere execution registration whenever
   an activated session ends.
7. Praebere releases its execution lock and owned residency only after its final active
   execution registration has been released.
8. Pontis releases the runtime-model reservation only after the final session holding
   that reservation has ended.

Manual orphan cleanup is an explicit, observable recovery operation. M0.1 does
not pretend that lack of conversational activity proves client disappearance.

### 5.1 Per-Session Provider Tools Control

Pontis must support a session-scoped Provider tools setting. The setting
controls whether Pontis injects ACP-discovered tools into the provider request
for that session.

The required behaviour is:

- Provider tools are enabled by default for a newly established session.
- When Provider tools are enabled, Pontis may discover ACP tools and attach
  `tools` / `tool_choice` declarations to the provider request.
- When Provider tools are disabled, Pontis must still apply the authoritative
  Praebere-selected model but must forward the provider request without
  `tools` or `tool_choice`.
- Disabling tools must not change the selected model.
- Disabling tools must not release the active execution session.
- Disabling tools must not prevent later re-enabling tools for the same active
  session, subject to normal provider/model compatibility.
- The setting is Pontis session state and must be visible through Pontis session
  inspection.

This allows a model such as `gemma3:4b` to be used through Rogare even when the
underlying provider rejects tool declarations for that model. The model remains
Praebere-selected and Pontis-owned; only provider-tool injection changes.

### 5.2 Rogare Provider Tools Control

Rogare must expose the Pontis Provider tools setting in its session UI.

The UI must:

- show whether Provider tools are currently on or off for the active session;
- allow the user to toggle Provider tools for the active session;
- update the displayed transport mode so the user can distinguish
  `HTTP + ACP tools` from `HTTP only`;
- preserve the active session when the toggle succeeds; and
- report toggle failures as session/control-plane errors, not model responses.

The toggle is session-scoped. Starting a new Rogare session returns to the
default Provider tools setting unless Pontis later defines persisted client
preferences.

### 5.3 Provider Tool-Rejection Cleanup

Some provider/model combinations reject requests that contain tool declarations.
For example, an Ollama-backed model may return a provider error equivalent to:

```text
does not support tools
```

When Rogare receives a fatal provider tool-declaration rejection for its active
session, Rogare must:

1. display the provider rejection clearly in the conversation UI;
2. request Pontis to end the active Rogare session;
3. release the Praebere execution registration through the normal Pontis
   session-end path;
4. mark the Rogare session as ended locally;
5. disable further message sending on the failed session; and
6. instruct the user to start a new session with Provider tools switched off
   for that model.

This prevents a failed tool-declaration request from leaving Rogare attached to
a poisoned or unusable provider session.

### 5.4 Why Client Heartbeat Is Not an M0.1 Requirement

The earlier heartbeat proposal assumed that Pontis could initiate an HTTP POST
to the external client after the client-initiated request had completed. The
normal client-to-Pontis request address and Moderari's response-stream heartbeat
do not establish that the client exposes a stable callback endpoint.

Requiring such a receiver would add client registration, networking,
authentication, race and timeout semantics without improving Pontis's authority
over session closure. The agreed explicit-control design is smaller, clearer and
auditable. Heartbeat-based automatic liveness detection is therefore removed
from the M0.1 design rather than retained as a competing implementation path.

### 5.5 Atomic Runtime-Model Reservation

M0.1 permits one authoritative runtime-global model reservation. Pontis must
apply the following operation atomically:

1. If no reservation exists, provisionally reserve the requested model for the
   selecting session.
2. If the same model is already reserved, add the selecting session to that
   reservation.
3. If a different model is reserved, reject the request before invoking
   Praebere.
4. If Praebere rejects the provisional selection, cancel the provisional
   reservation.
5. Release a session's reservation when that session closes, and release the
   global reservation when its final reserving session closes.

Concurrent requests for different models must have one deterministic atomic
winner; they must never leave two successful reservations. Conflict responses
must identify the authoritative reserved model and explain that all sessions
reserving it must end before a different model can be selected.

Clients must update their displayed model only from Pontis-confirmed state. A
requested model that lost or failed selection must never remain displayed as
though it were authoritative. The last selected model may remain in Praebere's
reported history after reservation release, but that historical value is not a
reservation and does not imply that the model is resident.

### 5.6 Demand Residency and External Ollama Boundary

Ollama is external infrastructure. Praebere must verify that its configured
Ollama endpoint is available during lifecycle startup, but Lumen must never
start or stop the Ollama process. Endpoint unavailability is a lifecycle-start
failure and Servire must roll back the services started by that stack-start
attempt.

Model selection records intent only. On a session's first ask, Praebere must:

1. inspect Ollama residency;
2. load the selected model only if it is not already resident; and
3. record residency ownership only if Praebere issued that load.

Concurrent active sessions using the reserved model share its residency. On
final active-execution release, Praebere unloads the model only when it owns the
residency. A model already resident before Lumen used it is not Lumen-owned and
must remain loaded. A failed owned-model unload is degraded partial completion;
it does not authorise Lumen to stop Ollama.

---

## 6. Pontis Session-Management UI

Pontis must gain a compact service-owned UI embedded in Servire, consistent with
the existing Nuntius and other component views. It must show enough state for an
operator to understand why Praebere remains locked and to close an abandoned
session deliberately.

For every session, display at least:

| Field | Requirement |
|---|---|
| Session identity | Authoritative Pontis `session_id` |
| Origin | Rogare, Replay, external client or other known origin |
| Lifecycle state | Established, active execution, request in flight, closing, closed, release pending, or operator-identified orphan |
| Model state | Selected runtime model associated with execution, when active |
| Created/updated time | Creation and most recent authoritative activity/change times |
| Request state | Whether work is currently in flight |
| Praebere state | Whether an execution registration exists and whether release is pending or failed |
| Provider tools | Whether Pontis will inject ACP tools for the session |
| Closure evidence | Closure time, reason and initiating origin when terminal |

The UI must provide:

- **Refresh**;
- **End Session** for normal graceful termination;
- **Force Close / Clear Orphan** for authorised recovery when graceful closure
  cannot complete; and
- explicit confirmation before a force-close operation.

The UI must not imply that an idle session is orphaned automatically. For M0.1,
orphaned is an operator determination supported by the displayed evidence.

---

## 7. Pontis Control Commands

Pontis must expose control-plane commands equivalent to the UI actions.

### 7.1 Client-Scoped Commands

```text
\obt pontis session status
\obt pontis session end
\obt pontis session new
\obt pontis session tools status
\obt pontis session tools on
\obt pontis session tools off
```

- `session status` reports the caller's authoritative session and execution
  state.
- `session end` idempotently ends only the caller's session.
- `session new` ends the caller's current session before creating and returning
  a replacement session identity.
- `session tools status` reports whether Pontis will inject ACP tools for the
  caller's current session.
- `session tools on` enables provider-tool injection for the caller's current
  session.
- `session tools off` disables provider-tool injection for the caller's current
  session while preserving the authoritative selected model.
- An ordinary client must not name or terminate another client's session.

### 7.2 Administrative Commands

```text
\obt pontis sessions
\obt pontis session end <session_id>
\obt pontis session force-close <session_id>
```

- `sessions` lists authoritative session and execution state.
- administrative `session end` requests the normal graceful closure path.
- `force-close` is restricted, confirmed where interactive, and reserved for
  orphan recovery or failed normal termination.
- Every command returns explicit before/after state and the Praebere release
  result.

All commands travel through the control plane and remain outside model context
and Vestigare conversational Trace evidence.

---

## 8. Authorisation, Audit and Failure Rules

- Self-service commands are scoped to the authenticated/current Pontis session.
- Cross-session listing, closure and force-close require an authorised
  administrative or trusted Servire origin.
- Closure records the request identity, origin, reason, time and whether it was
  graceful or forced.
- A force-close must never close unrelated sessions.
- Pontis must reject or explicitly defer normal closure while work is in flight,
  unless an authorised force-close policy deliberately overrides that safety
  boundary.
- Failure to release Praebere after local closure produces `release pending` or
  equivalent observable state and must be retryable/reconcilable.
- Repeating an end or release operation is idempotent.
- Starting a replacement session must not reuse the closed `session_id`.

---

## 9. Required Pontis Termination Behaviour

Every UI or command-driven Pontis session termination must:

1. prevent new work from entering the session while it is closing;
2. avoid closing a session with demonstrably in-flight work;
3. close the Pontis session and any associated ACP/provider binding;
4. remove session-scoped capability, backchannel and tool-routing state;
5. send the Praebere execution-release command through Nuntius if the session
   was activated;
6. remove the session from Pontis's active execution set;
7. leave unrelated sessions untouched;
8. record the closure reason and terminal time; and
9. behave idempotently if the session or execution registration has already
   been released.

If the final active execution session is released successfully, Praebere must
report its execution lock as released and unload any Praebere-owned residency.
If another active execution session remains, Praebere must retain both. Pontis
must independently retain the model reservation while any session still reserves
that model.

---

## 10. Praebere Persistence, Restart Reconciliation and Reset

Praebere stop, start, restart or unexpected process failure must not silently
discard the runtime-global model selection or the active execution-session state.
Otherwise restarting Praebere would bypass the N9.5 execution lock even though
Pontis still considers one or more sessions active.

Praebere must persist its active operational state whenever that state changes.
The persisted representation must include at least:

```text
provider identity
selected model
preferred model where it forms part of reported runtime state
active execution session IDs
selection-lock state
Praebere-owned residency evidence
state generation/version
last-updated time
```

The persisted Praebere `selection_locked` value provides execution-recovery evidence,
but Praebere's authoritative execution lock must be derived from the reconciled active
execution-session set. It is distinct from Pontis's pre-execution model reservation:

```text
selection_locked = active_execution_session_ids is not empty
```

State updates must be atomic or otherwise resilient to interruption. Praebere
must not accept an activation or release, report success and then leave only the
previous state recoverable after a crash.

### 10.1 Restart Recovery Sequence

On startup or restart, Praebere must:

1. load its most recently committed operational state;
2. restore the selected provider/model identity;
3. restore the persisted active execution-session registrations;
4. enter an explicit `reconciliation_pending` state when persisted active
   sessions exist or when authority cannot yet be established;
5. remain fail-safe and locked while reconciliation is pending if the persisted
   state contains active execution sessions;
6. obtain Pontis's authoritative current session/execution state through the
   agreed control path;
7. remove registrations that Pontis confirms are no longer active;
8. distinguish `release pending` sessions from live active executions, remove
   each pending release from Praebere's active set and acknowledge it to Pontis;
9. restore any valid active registration known to Pontis but missing from the
   recovered Praebere state;
10. recalculate the execution-session count, execution lock and owned-residency
    requirement; and
11. persist and expose `reconciliation_state: complete` only after both sides
    have acknowledged the reconciled result and before normal model-selection
    changes are accepted.

Praebere must not silently unlock merely because it restarted or because Pontis
is temporarily unavailable. Equally, it must not retain a stale persisted lock
indefinitely after Pontis has authoritatively confirmed that the sessions no
longer exist.

### 10.2 Authority and State Ownership

The restart design must preserve the existing authority boundary:

- Pontis determines which Lumen sessions and execution sessions are currently
  valid and which sessions reserve the runtime-global model;
- Praebere persists and enforces its model execution and residency state;
- Nuntius carries the reconciliation/control exchange;
- Servire coordinates managed service lifecycle but does not become the owner of
  conversational session state;
- configuration remains an initial/preferred-state input and must not overwrite
  recovered authoritative runtime state on every restart.

The reconciliation contract must include enough generation or instance identity
to distinguish current Pontis state from an old response produced before either
service restarted.

### 10.3 Managed Stop and Full-Stack Shutdown

During an orderly managed shutdown:

- Pontis should close/release active sessions before Praebere stops;
- Praebere must persist every accepted release;
- Praebere must not manufacture an unlocked state merely because its process is
  stopping;
- Praebere unloads only model residency that it caused;
- Lumen never starts or stops Ollama; and
- if shutdown is interrupted, the next Praebere start must recover and reconcile
  the last committed state.

Stopping or restarting Praebere alone while Pontis remains operational must
preserve the lock for every still-active execution session.

### 10.4 Administrative Execution-State Reset

Praebere requires a deliberately scoped control command for recovery from
orphaned execution registrations:

```text
\obt praebere execution reset
```

The safe reset must:

- query/reconcile with Pontis first;
- succeed only when Pontis confirms that no active execution sessions remain;
- clear Praebere's active execution-session set;
- clear the derived selection lock;
- preserve the selected model;
- preserve the preferred model;
- preserve provider identity, readiness and residency-ownership state;
- persist the reset immediately;
- return explicit state before and after the operation; and
- record the origin, request identity, reason and time of the reset.

The safe reset must be rejected while Pontis reports any live active execution
session. It must never become an ordinary way to override the N9.5 model lock.

### 10.5 Forced Recovery Reset

An emergency recovery form may be required when Pontis is unavailable or its
state cannot be reconciled:

```text
\obt praebere execution reset force
```

If implemented, the forced reset must:

- be restricted to an authorised administrative origin;
- clearly state that the normal Pontis authority check was overridden;
- clear only execution-session registrations and the derived lock;
- preserve selected/preferred model and provider residency-ownership state;
- persist the result immediately;
- emit conspicuous operational/audit evidence; and
- require subsequent reconciliation before new execution activation where
  necessary to prevent an old Pontis instance from silently restoring stale
  registrations.

Whether M0.1 exposes the forced form to an external researcher or retains it as
a Servire recovery operation must be decided explicitly during implementation.

A broad command such as `\obt praebere reset` should not be introduced without
separately defining whether it clears selection, unloads a model or changes residency
ownership. Those operations are materially different
from repairing the execution-session lock.

### 10.6 Restart and Reset Failure Cases

The implementation must define and expose:

- corrupt or unreadable persisted state;
- partially committed state after process interruption;
- Praebere starting before Pontis is available;
- Pontis restarting while Praebere retains registrations from the prior Pontis
  instance;
- failed or timed-out reconciliation;
- a session ending while reconciliation is in progress;
- new execution activation attempted while reconciliation is pending;
- reset attempted during an in-flight execution;
- failure to persist a reset that was otherwise accepted; and
- recovery after the reset response was lost but the reset was committed.

Success must never be inferred from missing state, silence or failure to contact
Pontis. Every unresolved recovery condition must be observable rather than
appearing as a normally unlocked runtime.

---

## 11. Required Acceptance Tests

### 11.1 Praebere Lock Counting

- Establish sessions without model interaction and confirm they do not lock the
  model.
- Activate one execution session and confirm the model locks.
- Activate two execution sessions and confirm the active count is two.
- End one and confirm the model remains locked with count one.
- End the final execution session and confirm the model unlocks.

### 11.2 Rogare

- End Session closes the Pontis session and releases Praebere.
- New Session ends the prior session before creating the replacement.
- Rogare stop ends its current session.
- Rogare restart performs the same stop/end lifecycle.
- Ending Rogare does not affect an unrelated Pi, Replay or external session.
- Rogare exposes a Provider tools toggle for the active session.
- Toggling Provider tools off switches the active session to provider requests
  without ACP tool declarations while preserving the selected model.
- Toggling Provider tools on restores normal ACP tool declaration injection for
  the active session.
- Rogare displays whether the active transport mode is `HTTP + ACP tools` or
  `HTTP only`.
- A fatal provider tool-declaration rejection ends the active Rogare session and
  instructs the user to start a new session with Provider tools disabled for the
  incompatible model.
- Investigate abnormal Rogare termination and verify or define recovery.

### 11.3 Pontis UI and External-Client Commands

- Pontis UI lists established and active execution sessions distinctly.
- The displayed Praebere registration and release state explains why model
  selection is locked or why release is pending.
- An external client can inspect and idempotently end its own session.
- An external client can inspect and toggle its own Provider tools setting.
- An external client cannot inspect or end an unrelated client's session.
- `session new` ends the current session before returning a fresh `session_id`.
- Normal End Session refuses or deterministically defers closure while work is
  in flight.
- An authorised operator can gracefully end a named session.
- An authorised operator can confirm and force-close an orphaned session.
- An unauthorised origin cannot list all sessions, end a named foreign session
  or force-close any session.
- Closing one client session releases only that session's activated Praebere
  registration.
- Disabling Provider tools removes provider `tools` and `tool_choice` from
  forwarded requests without changing the Praebere-selected model.
- Enabling Provider tools restores normal ACP tool injection for compatible
  provider/model combinations.
- A failed Praebere release remains visible as release pending and succeeds on
  retry or reconciliation without duplicating closure.
- Force Close reports local Pontis closure and pending Praebere release as
  partial success, not as either complete success or an undifferentiated failure.
- Session-state labels use human-readable display text such as `release pending`
  while preserving machine-state identifiers in the API.
- UI and control traffic does not enter model context or Vestigare Trace.
- Idle time alone never labels or closes a session as orphaned.

### 11.4 Restart, Persistence and Reconciliation

- Pontis restart does not leave unreleasable phantom Praebere sessions.
- Praebere restart with one active execution session restores the selected model
  and remains locked through reconciliation.
- Praebere restart with multiple active execution sessions restores every valid
  registration and the correct count.
- Praebere restart with only established/non-executing Pontis sessions remains
  unlocked.
- Stopping/restarting Praebere alone does not release active Pontis sessions.
- Releasing one recovered session leaves the lock active while another remains.
- Releasing the final recovered active execution session unlocks selection.
- A stale persisted execution registration is removed only after Pontis
  reconciliation confirms it is no longer active.
- A session closed while Praebere is stopped remains `release pending`; after
  Praebere starts, reconciliation removes it from Praebere's active set,
  acknowledges release to Pontis and reaches `complete` without requiring a
  further client ask.
- A Pontis-active session missing from recovered Praebere state is restored by
  reconciliation.
- Praebere remains fail-safe while Pontis is unavailable and persisted active
  registrations exist.
- Model selection is rejected while reconciliation is pending.
- Failed execution release is observable and recoverable.
- Corrupt or interrupted persisted state produces an explicit recovery failure
  rather than a silently unlocked runtime.
- Repeated activate/end cycles do not leak sessions or retain the model lock.

### 11.5 Model Reservation, Residency and Ollama Boundary

- Praebere lifecycle startup succeeds when the configured Ollama endpoint is
  available and fails explicitly when it is unavailable.
- An Ollama-unavailable Praebere startup causes Servire to roll back services
  started by that stack-start attempt.
- Praebere and the Lumen stack never start or stop Ollama.
- Starting Praebere with no selected model does not load a model.
- Selecting a model records selection and reservation but does not load it.
- The first ask loads the selected model when it is absent and records that
  Praebere owns the resulting residency.
- If the selected model was already resident, first ask uses it without claiming
  residency ownership.
- A second session may reserve and execute with the same model.
- A second session requesting a different model receives a human-readable
  conflict before Praebere selection is called.
- Two concurrent first selections for different models produce exactly one
  authoritative reservation.
- A failed Praebere selection cancels its provisional reservation.
- Clients display Pontis-confirmed authoritative state after both successful and
  rejected selections; a rejected request does not alter the displayed model.
- Ending one of several reserving sessions retains the reservation; ending the
  final reserving session releases it.
- Ending the final active execution unloads the model only when Praebere caused
  the residency; a pre-existing resident model remains loaded.
- An unload failure is reported as degraded partial completion and does not stop
  Ollama.

### 11.6 Administrative Reset

- Safe execution reset succeeds when Pontis confirms no active execution
  sessions.
- Safe execution reset is rejected while Pontis reports a live active execution
  session.
- Safe reset preserves selected model, preferred model, provider identity and
  residency-ownership state.
- Safe reset persists its result and survives Praebere restart.
- Safe reset returns and records state before and after the operation.
- If implemented, forced reset is restricted, explicit and auditable.
- Forced reset clears only execution registrations/lock and does not silently
  stop Ollama, unload model residency or change the selected model.
- A reset racing with execution activation or release has a deterministic atomic
  outcome.
- Lost reset responses can be reconciled without applying an unsafe duplicate
  operation.

---

## 12. Completion Boundary

This lifecycle work is complete when:

- Pontis remains the sole session-lifecycle authority;
- Praebere unlocks only when the final active execution session has ended;
- Pontis atomically reserves one runtime-global model and prevents concurrent
  clients from successfully selecting conflicting models;
- Praebere loads the reserved model only on first ask and unloads it on final
  execution release only when Praebere caused that residency;
- Ollama remains externally operated: Praebere checks availability but Lumen
  never starts or stops it;
- Rogare explicitly ends its session through End Session, New Session, stop and
  restart paths;
- Pontis provides an embedded session-management UI that exposes authoritative
  session, execution and Praebere-release state;
- external clients can inspect, end and replace their own Pontis session through
  client-scoped control commands;
- clients can inspect and toggle per-session Provider tools injection without
  changing the selected model or session ownership;
- Rogare exposes that Provider tools setting and accurately distinguishes
  `HTTP + ACP tools` from `HTTP only`;
- provider/model combinations that reject tool declarations can still be used
  through Rogare by starting a session with Provider tools disabled;
- fatal provider tool-declaration rejection ends the affected Rogare session and
  releases its Praebere execution registration through Pontis;
- authorised operators can gracefully end or explicitly force-close an
  abandoned/orphaned session without affecting unrelated sessions;
- normal closure protects in-flight work and forced recovery is restricted,
  confirmed and audited;
- Praebere persists selected-model and active execution-session state on every
  accepted state change;
- Praebere restart does not silently unlock or forget still-active sessions;
- recovered Praebere state is reconciled against Pontis authority before normal
  selection changes resume;
- reconciliation processes deferred releases automatically on Praebere restart
  and reaches `complete` without requiring a subsequent client request;
- safe execution-state reset clears only orphaned execution registrations after
  Pontis confirms that no live active execution sessions remain;
- any forced recovery path is explicitly authorised, scoped and auditable;
- all normal, failure and restart paths reconcile Pontis and Praebere state;
- an abandoned session can be identified and cleared authoritatively so that it
  cannot leave the runtime-global model permanently unrecoverable;
  and
- all session-lifecycle traffic remains outside conversational execution and
  Vestigare Trace evidence.

Automatic client-heartbeat detection is not part of this N9.5 completion
boundary. It may be reconsidered only as a separate future capability with a
defined callback or lease contract; it is not required for M0.1 release.


---

## 12. M0.1 Closeout Note — 2026-09-04

This lifecycle document is on the M0.1 critical path through N9.6.3 only.

Remaining closure work:

- correct the known N9.6.3 defects identified in the September 4 live lifecycle pass;
- rerun the complete cross-client/session/start/stop lifecycle acceptance sequence;
- capture final N9.6.2 reconciliation evidence; and
- confirm all modified-service quality gates remain clean.

The full provider-neutral readiness taxonomy described separately by N9.6.4 is a
**nice-to-have/post-M0.1 enhancement**. The compact Praebere operational UI remains
required for M0.1, but it may use the concrete provider availability, model
availability/residency, selection, lock, persistence and reconciliation state already
defined by this lifecycle work.
