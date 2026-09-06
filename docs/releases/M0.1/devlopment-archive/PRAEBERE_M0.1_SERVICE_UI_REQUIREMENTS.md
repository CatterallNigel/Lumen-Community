# Praebere M0.1 Service UI Requirements

**Status:** Complete and live-validated for M0.1  
**Target:** Lumen External Research Distribution M0.1 — N9.7 Praebere UI  
**Date:** 2026-09-05  
**Scope:** Praebere-owned provider/model operational UI exposed through Servire

---

## Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-09-01 | Nigel Catterall | Defined the compact M0.1 Praebere service UI, authority boundaries, model and execution-state presentation, guarded actions, Servire integration and acceptance requirements. |
| 1.1 | 2026-09-04 | Nigel Catterall | Reconciled the UI with the later N9.6 lifecycle decision: Ollama is external infrastructure, Praebere owns only model residency it causes, and the full N9.6.4 provider-neutral readiness taxonomy is optional post-M0.1. Kept the compact operational UI as an M0.1 requirement using concrete provider availability, model availability/residency, selection, lock, persistence and reconciliation state. |
| 1.2 | 2026-09-05 | Nigel Catterall | Reconciled the requirements with completed N9.6.3 behaviour and the deferred N9.6.4 contract. Clarified the Praebere-owned iframe boundary, Praebere authority for model selection/reservation/residency, available/reserved/locked states, Pontis `not active` semantics, and persisted guarded reconcile/reset operations. |
| 1.3 | 2026-09-05 | Nigel Catterall | Defined cache-only routine UI polling and explicit live discovery through Refresh Models. Added UI requirements for controlled activation/load failure, cancellation of only the requesting session's reservation, persistence of the resulting authoritative state and clear client/operator reporting. |

---

## 1. Purpose

Praebere owns provider discovery, model discovery, runtime-global model selection,
model reservation, execution-session lock state and model-residency ownership.
Ollama remains external infrastructure whose process lifecycle is outside Lumen.
These Praebere operations are available through the common Nuntius `\obt`
control path, but an operator should not need to type control commands for routine
Praebere inspection and administration.

M0.1 therefore requires a compact Praebere-owned UI, visually and operationally
consistent with the existing Nuntius and Moderari service surfaces and exposed in
the Servire portal.

The UI is not intended to become a general model-management application. It is an
operator surface for the authoritative Praebere state and bounded M0.1 actions.

---

## 2. Implementation Timing

The Praebere UI is the **N9.7** implementation following completion of N9.6.3. It
must consume the implemented N9.6 state semantics rather than introducing a
competing or temporary representation.

The UI must consume the concrete N9.6 persistence, reconciliation, provider-availability
and model-residency state that has landed. The full N9.6.4 provider-neutral readiness
taxonomy is optional post-M0.1 and must **not** block this compact UI.

UI implementation must not redefine provider availability, model residency ownership,
persistence, restart reconciliation or reset behaviour merely to make the display
convenient.

---

## 3. Authority and Ownership Boundary

Praebere owns and serves the complete UI, including its HTML, styling, JavaScript,
API calls, state interpretation, actions and error presentation. Servire exposes
that same page in the Praebere service tab through an iframe and may provide an
**Open directly** action, following the existing service-owned UI pattern.

```text
Servire Praebere tab / Open directly
    → same Praebere-owned UI
    → Praebere management/read APIs
    → authoritative Praebere runtime state
```

The following boundaries apply:

- Praebere is authoritative for model discovery, selection, reservation, residency
  ownership and derived `available`/`reserved`/`locked` state.
- Pontis remains the session-lifecycle authority.
- Nuntius remains the common cross-service and external-client control path.
- Servire remains the managed-service lifecycle and portal authority.
- Rogare retains its session-establishment model dropdown.
- The Praebere UI must not query Ollama directly from the browser or Servire.
- Servire must not reproduce Praebere business logic or maintain a competing model
  catalogue.
- Praebere's UI must work when opened directly and must not depend on Servire-side
  JavaScript or duplicated Servire state.
- The UI must use the same Praebere internal operations and validation rules as the
  Nuntius/`\obt` path, even where it calls native Praebere management APIs rather
  than constructing conversational commands.

---

## 4. Relationship to `\obt` and Rogare

The UI supplements but does not replace the Praebere control commands.

The `\obt praebere` commands remain required for:

- supported external clients;
- Fiducia and Repetere orchestration;
- automation;
- diagnostic and acceptance testing; and
- the common Nuntius control-plane contract.

The interfaces serve different purposes:

| Interface | Purpose |
|---|---|
| Praebere UI | Installation-wide provider/model state, diagnostics and bounded administration |
| Rogare model selection | Model choice for an established session when runtime-global selection is available |
| `\obt praebere ...` | External-client, automation and common control-plane operation |

All three must converge on the same authoritative Praebere state.

---

## 5. Servire Navigation

Praebere must appear as its own service tab in Servire. The established ordering
rule remains:

```text
Servire
Fiducia
Moderari
Nuntius
Pontis
Praebere
Repetere
Rogare
Vestigare
```

The tab must:

- show the normal service availability state;
- embed the Praebere-owned UI;
- offer **Open directly** where the existing service UI pattern provides it;
- preserve direct/open-service navigation;
- avoid duplicating normal Servire start/stop/restart controls inside the embedded
  Praebere surface; and
- avoid filling the Servire Operational Log with successful UI refresh traffic.

---

## 6. Overview

The top of the Praebere UI should present a compact authoritative summary. M0.1
requires at least:

| Summary | Example |
|---|---|
| Provider | `Ollama — Running` |
| Provider lifecycle | `External infrastructure` |
| Selected model | `qwen2.5-coder:14b` or `None` |
| Selection state | `Available`, `Reserved`, `Locked` or `Reconciliation pending` |
| Reservations | Count of open Pontis sessions reserving the selected model |
| Active executions | Count of active execution sessions |
| Model residency ownership | `Praebere-loaded`, `Pre-existing/external`, or `Not resident` where known |

Where available without crowding the summary, also expose:

- preferred model;
- provider endpoint;
- last successful discovery refresh;
- persistence/reconciliation health; and
- concise provider or state failure indication.

Model availability in this surface represents Praebere's catalogue from the last
successful startup or explicit discovery refresh. Routine UI polling must not imply
that Ollama discovery is continuously repeated.

The overview must never collapse provider reachability, model availability,
model residency, selection and execution lock into one generic `running` indicator.
N9.6.4 readiness is deferred to Future Development and is not required by this UI.
If implemented later, its readiness state must remain distinct as an extension.

---

## 7. Models

The UI must provide a model table derived from authoritative Praebere discovery and
runtime state.

| Field     | Requirement                                               |
| --------- | --------------------------------------------------------- |
| Model     | Provider-reported model identity                          |
| Available | Whether the model is locally available                    |
| Resident  | Whether the provider currently reports it loaded/resident |
| Preferred | Whether it matches the configured preference              |
| Selected  | Whether it is the authoritative runtime-global selection  |
| Action    | Select/current/disabled state                             |

### 7.1 Selection Behaviour

- Model selection is initiated by a client for an authoritative Pontis session. The Praebere UI displays the resulting authoritative selection and reservation state but does not initiate, change or clear ordinary model selection. Returning Praebere to no selection is available only through the guarded Reset Runtime State operation.

### 7.2 State Language

The labels **available**, **resident**, **preferred**, **selected**, **reserved** and
**locked** must retain their distinct N9 meanings:

- `available`: no open session reserves the selected model;
- `reserved`: one or more open sessions reserve the model, with no active model
  execution;
- `locked`: one or more sessions are active in model execution; and
- `reconciliation_pending`: Praebere cannot yet safely establish the authoritative
  reservation/execution state.

Colour may reinforce state, but colour alone must not carry the distinction. If the
deferred provider-neutral readiness contract is implemented later, **ready** remains
a separate state.

---

## 8. Reservations and Active Execution Sessions

Praebere must display both the open-session reservations that prevent selection of
a different model and the active execution registrations that produce the
runtime-global model lock.

At minimum show:

| Field       | Requirement                                              |
| ----------- | -------------------------------------------------------- |
| Session ID  | Authoritative Pontis `session_id`                        |
| Origin/type | Rogare, external client, Replay or other known origin    |
| Model       | Effective runtime-global model                           |
| Reservation | Reserved or not reserved                                 |
| Execution   | Active or not active in model execution                  |
| Activated   | Activation time where recorded                           |
| State       | Reservation, execution and reconciliation/recovery state |

Praebere may display and diagnose these registrations, but it must not become the
normal authority for terminating them. Normal session closure remains a Pontis
operation and is initiated through Rogare End Session, external-client lifecycle,
Replay cleanup or the appropriate Pontis surface.

The UI must not use Pontis `not active` as shorthand for a closed session or an
available model. An established session may be not active in model execution while
still reserving the selected model. The UI should explain separately why selection
is reserved or locked and identify the responsible registrations. Where a Pontis
inspection surface exists, the UI may link to it rather than duplicate session
management.

---

## 9. Runtime, Persistence and Reconciliation

The UI must expose the recovery state defined by
`PONTIS_SESSION_LIVENESS_AND_PRAEBERE_MODEL_LOCK_RELEASE.md` without requiring the
operator to inspect logs.

Show where available:

- persisted-state generation/version;
- last persisted update time;
- reconciliation state;
- Pontis connectivity;
- recovered active-execution count;
- reconciled active-execution count;
- reserved-session count and identifiers;
- last successful reconciliation;
- persistence, recovery or reconciliation failure; and
- whether model selection is disabled because reconciliation is pending.

The UI must not present `reconciliation_pending`, corrupt persisted state or failed
authority reconciliation as a normally available runtime. Reconciliation outcomes,
including pending or failed outcomes, must update `praebere_runtime_state` through
Praebere's normal atomic state-transition and persistence path.

---

## 10. M0.1 Actions

The bounded Praebere UI actions are:

### 10.1 Refresh Models

- Explicitly request live provider/model discovery through Praebere; this is the only
  routine UI action that replaces the cached model catalogue from Ollama.
- Refresh provider availability and model availability/residency state, then store the
  resulting catalogue and last-successful-refresh time as Praebere's authoritative
  cached discovery state.
- Do not alter selection merely because provider reality changed.
- Report provider/discovery failure explicitly.
- On failure, retain the previous catalogue only with a clear stale/failed-refresh
  indication.

### ~~10.2 Select Model~~

- ~~Use the authoritative Praebere selection operation.~~
- ~~Apply all N9.5 lock, validation and idempotency rules.~~
- ~~Return the effective state, not merely the requested value.~~

### 10.3 Reconcile with Pontis

- Request the defined Pontis/Praebere reconciliation operation.
- Display progress and terminal outcome.
- Preserve fail-safe locking while authority remains unresolved.
- Do not manufacture or remove sessions only in browser state.
- Atomically persist the reconciled selected model, reserved-session IDs,
  active-execution IDs, derived selection state, residency ownership,
  reconciliation outcome, generation/version and update timestamp in
  `praebere_runtime_state`.
- Report success only after the reconciled runtime state has been persisted.

### 10.4 Reset Runtime State

- Expose a guarded Praebere runtime-reset operation without restarting the service.
- Require confirmation.
- Reconcile with Pontis as part of the guard and disable or reject reset while Pontis
  reports any open/reserving session or active execution session.
- Retain the preferred model and provider configuration.
- Clear the selected model, reserved-session IDs and active-execution IDs.
- Release Praebere-owned model residency before clearing residency ownership; leave
  pre-existing/external residency untouched.
- Set the resulting selection state to `available` and reconciliation state to
  `complete` only after every guard and required transition succeeds.
- Atomically persist the clean result, reset reason, generation/version and update
  timestamp in `praebere_runtime_state` through the same state-transition path used
  by client-driven changes.
- Display state before and after the operation.
- Record the origin, request identity, reason and time.
- Report success only after runtime mutation and persistence both succeed. A failure
  or partial outcome must remain explicit and fail-safe.

M0.1 exposes both the guarded Reset Runtime State action and a forced recovery reset. The forced reset must be separately authorised, require explicit confirmation, be visually distinguished as a hazardous recovery action, and clearly describe the state that will be cleared and the risks of proceeding. Its invocation and outcome must be recorded with the operator, reason, request identity and timestamp. It must never appear as an ordinary routine action or be triggered implicitly.

---

## 11. Actions Explicitly Outside the UI Boundary

The M0.1 Praebere UI must not:

- terminate ordinary Pontis sessions as a normal action;
- edit Moderari model profiles or context-window policy;
- manage prompts, conversations, Trace, Replay or assessment;
- provide per-session model selection;
- query or control Ollama directly from browser-side code;
- change provider ownership implicitly;
- silently unload the selected model;
- silently stop externally owned Ollama;
- bypass the guarded `Reset Runtime State` operation with an unvalidated reset; or
- duplicate Servire's normal service start/stop/restart interface.

---

## 12. Refresh and Error Behaviour

- Poll or refresh often enough to show changes made through Rogare, external clients,
  Repetere/Fiducia or `\obt` without a manual browser reload.
- Periodic UI polling must read Praebere's cached catalogue and authoritative runtime
  state only. It must not query Ollama or initiate provider discovery.
- Use bounded polling consistent with the existing M0.1 UI pattern. Routine successful polling must not be written to the Servire Operational Log or normal Praebere operational logs. Record only polling failures, recovery after failure, material state changes and state-changing operator actions. Repeated identical failures must be rate-limited or consolidated to prevent log flooding.
- ~~Stop or reduce polling when the UI is not active where practical.~~
- Ensure refresh traffic does not enter conversational Trace.
- Ensure routine successful refresh traffic does not dominate Servire Operational
  Logs.
- Preserve the last known state only with a clear stale/unavailable indication after
  refresh failure.
- Distinguish operation failure from provider unavailability, model
  availability/residency failure, reserved selection, locked selection and
  reconciliation failure.
- When model activation fails because Ollama cannot load the selected model, display a
  controlled explanation that the requesting session was not activated and its model
  reservation was cancelled. Display the resulting authoritative state as `available`
  when no reservation remains, or `reserved` when other sessions still reserve the
  selected model.
- Do not display raw internal exception data or excessively nested JSON as the normal
  operator message.

---

## 13. Security and Control Boundaries

- Use the existing M0.1 service access and Servire embedding boundary.
- Do not expose model-residency or recovery operations beyond the intended host
  entry points.
- Apply the same validation and authorization to direct UI operations as to their
  control-plane equivalents.
- Protect reset/recovery operations from accidental or unauthorised invocation.
- Preserve request identity and operational evidence for state-changing actions.
- Keep state-changing UI operations outside conversational model context and
  Vestigare Trace turns.

---

## 14. Acceptance Scenarios

At minimum validate:

1. Praebere appears between Pontis and Repetere in the established Servire tab
   ordering.
2. The embedded UI and **Open directly** surface use the same Praebere-owned page and
   show the same authoritative state.
3. The UI operates directly without depending on Servire-side JavaScript or duplicated
   Servire state.
4. Provider state identifies Ollama as external infrastructure.
5. Model-residency ownership distinguishes Praebere-loaded residency from
   pre-existing/external residency.
6. The model table matches authoritative Praebere/Ollama discovery.
7. Available, resident, preferred, selected, reserved and locked states remain
   distinct.
8. The Praebere UI does not provide ordinary model-selection actions.
9. A model selected through Rogare, Repetere or a supported external client appears
   without a manual Praebere-page refresh.
10. An invalid or conflicting client selection leaves the previous authoritative
    selection and reservation state unchanged in the UI.
11. After client selection, the selecting open session is shown as reserving the
    model before its first ask.
12. First ask changes the applicable session from reserved/not-active to active model
    execution, and the UI shows the reservation and execution facts separately.
13. An established Pontis session shown as `not active` is not presented as closed or
    as proof that model selection is available.
14. Ending one of several active execution sessions updates the active count while
    preserving the reservation or lock required by remaining sessions.
15. Ending the final active execution removes the execution lock, but selection
    remains reserved while any open reserving session remains.
16. Ending the final reserving session changes the authoritative selection state to
    available.
17. Praebere restart restores selected, reserved and active state from
    `praebere_runtime_state` and visibly enters and exits reconciliation.
18. Pontis being unavailable during reconciliation remains visibly fail-safe rather
    than appearing available or unlocked.
19. **Refresh Models** updates provider reality without silently changing model
    selection.
20. Ordinary UI polling, `\obt praebere models` and model-selection validation use
    Praebere's cached catalogue and do not query Ollama.
21. A model installed after startup appears only after a successful **Refresh Models**
    action replaces the cached catalogue.
22. If Ollama cannot load the selected model on first ask, the requesting session is
    not registered as active, only its reservation is cancelled, the corrected state
    is persisted, and the client receives the load failure and cancellation outcome.
23. A failed activation produces `available` when no other reservation remains and
    preserves `reserved` when another session still reserves the selected model.
24. **Reconcile with Pontis** updates the displayed runtime state and persists its
    terminal outcome in `praebere_runtime_state`.
25. Guarded **Reset Runtime State** is disabled or rejected while any open/reserving
    or active session exists.
26. With Pontis-confirmed zero live sessions, guarded reset clears selected, reserved
    and active runtime state, releases only Praebere-owned residency and persists the
    clean state.
27. Forced recovery reset is separately authorised, visually conspicuous and requires
    explicit confirmation.
28. Forced recovery reset clearly identifies the state being cleared and the risks of
    proceeding.
29. Every forced reset request and outcome records the operator, reason, request
    identity and timestamp.
30. Reconcile and reset outcomes survive a subsequent Praebere restart because they
    updated `praebere_runtime_state`.
31. Failed and partial actions display concise operator-facing explanations and do not
    present an unsafe state as successfully recovered.
32. UI polling and control activity remain outside model context and Vestigare
    conversational Trace.
33. Routine successful polling is absent from the normal Servire and Praebere
    operational logs.
34. Polling failures, recovery, material state changes and state-changing operator
    actions are recorded without flooding the logs with repeated identical failures.
35. Praebere, Servire and any other modified-service tests, Ruff and mypy checks pass.
---

## 15. Exit Condition

The M0.1 Praebere UI is complete when an operator can inspect authoritative provider
availability, model availability/residency, selection, persistence, reconciliation
reservation and execution-lock state and perform the bounded safe actions without typing `\obt`
commands, while external clients and automation continue to use the common Nuntius
control path.

The full provider-neutral readiness taxonomy is not required for M0.1. If implemented
later, it should extend this surface without redefining the concrete state above.

The UI must remain a presentation and operator-action surface over Praebere's
authoritative operations. It must not introduce competing state, bypass Pontis
session authority, obscure the runtime-global model lock or expand M0.1 into general
multi-model resource management.
