# Lumen M0.1 Replay — Agreed Implementation Sequence

**Status:** Agreed implementation baseline  
**Date:** 2026-09-07  
**Scope:** Vestigare, Repetere and the narrow Pontis lifecycle addition required for Replay. Fiducia remains deferred until Iteration 8 is discussed.

## Purpose

Implement the M0.1 Replay changes in independently testable iterations. Each iteration builds on a verified contract from the preceding iteration. Package-local preparatory work may be included when it avoids reopening the same model or API later, but later milestone behaviour must remain inactive until its own iteration and acceptance gate.

## Governing decisions

- Praebere is the authority for the single runtime-global model.
- Repetere reaches Praebere through Nuntius `\obt` commands and never controls Ollama directly.
- Vestigare owns Trace recordings, messages and recording-level provenance.
- Repetere owns Experiments, Runs, execution lifecycle and divergence results.
- Replace `replay_sessions` with `experiments` and `replay_runs` with `experiment_runs`; no data migration or backward compatibility is required.
- Existing test data in `trace_recordings`, `trace_messages`, `replay_sessions` and `replay_runs` may be cleared.
- `researcher_name` is retained in the Experiment schema with the M0.1 default value `M0.1`; no researcher-name UI is required.
- Unstage changes an Experiment to `UNSTAGED`; it does not delete the Experiment or its evidence.
- Every Run receives a genuinely new Pontis execution session ID. A completed, failed or recovered Run must not reuse a previous session ID or Moderari context.
- Replay-created evidence remains in Vestigare's existing Trace collections, explicitly classified and correlated to its source Trace, Experiment and Run.
- Terminal Run results and evidence are immutable. Post-terminal recovery may update cleanup/reconciliation fields only.
- The observed Moderari/Ollama model activation outside Praebere lifecycle is documented as an artifact and is not a separate M0.1 investigation. The new prerequisite flow prevents Repetere from using that path.

## Recommended implementation sequence

### Iteration 1 — Vestigare evidence and model-provenance contract

Implement authoritative recording-level model provenance in Vestigare.

At Trace start, Vestigare queries Praebere state through Nuntius and records the authoritative provider/model identity in `trace_recordings`. Define the optional child-Trace correlation fields required by Iteration 5 at the same time:

- `trace_kind`: `source` or `replay`
- `source_trace_id`
- `experiment_id`
- `run_id`
- execution `session_id`
- authoritative provider/model metadata

The correlation fields are preparatory schema/API capacity only in this iteration; Repetere does not yet create Replay child Traces.

**Acceptance gate:** A newly captured source Trace contains authoritative model provenance obtained through the supported Nuntius/Praebere path, and existing source-Trace recording behaviour remains correct.

### Iteration 2 — Experiment and Run persistence foundation

Replace the disposable Replay persistence model with clean `experiments` and `experiment_runs` collections. Implement the complete domain fields and indexes needed by Iterations 3–6 so persistence does not need repeated structural rewrites.

This includes:

- named Experiment records linked to a source Trace;
- `researcher_name: "M0.1"` by default;
- staged/unstaged Experiment lifecycle;
- append-only Run identity and Experiment relationship;
- fields for required model, Pontis session, child Trace, progress, result, error and cleanup/recovery state;
- terminal-result immutability rules;
- `UNSTAGED` behaviour without evidence deletion.

Later runtime fields may be present but remain unset until their owning iteration.

**Acceptance gate:** Stage, list, inspect and unstage operate against the replacement collections; Run records can be created and retrieved without invoking Replay execution.

### Iteration 3 — Fresh Pontis session lifecycle per Run

Add the narrow Pontis lifecycle contract required for a Repetere-owned Replay execution session, and integrate it into Repetere Run creation.

The session must be distinct from the `session-http-*` control-request session created when the Repetere UI submits the start command through Pontis. The Replay execution session must be registered with Pontis, receive a new ID for every Run, and be closed on the normal completion path.

Include basic idempotent close behaviour now so iteration testing cannot leak sessions. Full failure recovery remains Iteration 6.

**Acceptance gate:** Two consecutive Runs receive different Pontis execution session IDs; each normal completion closes its own session; Moderari does not reconnect or inherit context from the preceding Run.

### Iteration 4 — Model validation, reservation, activation and fail-fast rules

Implement the Repetere model prerequisite flow through Nuntius/Praebere:

1. Read the required provider/model from the source Trace.
2. Query authoritative Praebere state.
3. Validate model availability.
4. Reject a conflicting selected, reserved, resident or selection-locked model.
5. Select and reserve the required model for the Run's Pontis session.
6. Issue explicit `execution activate` and require positive acknowledgement.
7. Proceed to Moderari only after successful activation.

Add local compensating cleanup for every prerequisite failure introduced by this iteration. Repetere must release only its own session's reservation contribution. Full cross-service cleanup/recovery is completed in Iteration 6.

**Acceptance gate:** The required model is explicitly selected, reserved and activated before any Replay inference; unavailable or conflicting states fail before Replay traffic reaches Moderari, with a clear persisted error, UI result and operational log entry.

### Iteration 5 — Replay child Trace creation and correlation

Use the Vestigare contract prepared in Iteration 1 to create one Replay child Trace per Run. Record Replay messages in the existing Vestigare collections while distinguishing them from source recordings.

The child Trace must carry:

- `trace_kind: replay`;
- `source_trace_id`;
- `experiment_id`;
- `run_id`;
- the fresh Pontis execution session ID;
- authoritative model provenance.

**Acceptance gate:** A Run produces a separately identifiable child Trace whose evidence is fully correlated to the Run and original source Trace, without appearing as an unrelated top-level source recording.

### Iteration 6 — Terminal results, cleanup and recovery

Complete the Run state machine and all terminal paths, including matched, diverged, failed, cancelled/interrupted where supported, and recovered cleanup.

On every terminal path:

- persist the terminal result once;
- finalise the child Trace appropriately;
- close the Run's Pontis session;
- release only the Run's Praebere execution/reservation contribution;
- record cleanup outcome and any release-pending/recovery state;
- prevent later mutation of terminal result/evidence.

Recovery is idempotent and may repair cleanup state without rewriting the terminal scientific result.

**Acceptance gate:** Normal, divergence and induced-failure tests leave no unintended open Replay session, active execution or reservation; retrying cleanup is safe; terminal evidence remains unchanged.

### Iteration 7 — Repetere UI hierarchy and polling

Update the operator UI only after the backend lifecycle is stable. Present the hierarchy clearly:

- source Trace;
- Experiment;
- Run history;
- selected Run status/result;
- linked Replay child Trace evidence.

Polling and operation messages must distinguish accepted/running state from failures. In particular, remove the observed nonsensical persistent red message `Operation failed: replay running.` and ensure terminal state replaces transient start feedback.

Replay child Traces must not appear as unrelated source recordings in the top-level Trace list.

**Acceptance gate:** The UI accurately follows a Run from start through terminal state, supports repeated Runs under one Experiment, exposes correlation and cleanup failures, and does not display stale or contradictory operation banners.

### Iteration 8 — Fiducia scheduling, PID recovery and Clear Logs

Deferred. Define and agree this work with the Fiducia package and live runtime evidence before implementation.

### Iteration 9 — Cross-service and live acceptance testing

Run the final clean-start acceptance matrix across Vestigare, Repetere, Pontis, Nuntius, Praebere, Moderari and Servire, followed by Fiducia once Iteration 8 is defined.

At minimum, verify:

- capture of source Trace model provenance;
- staging, unstaging and repeated Runs;
- fresh session isolation;
- correct-model activation;
- model conflict and unavailable-model fail-fast behaviour;
- matched and divergent child Trace evidence;
- failure cleanup and restart recovery;
- UI hierarchy and polling;
- absence of model activation by Replay outside the Praebere-authorised path.

## Package-oriented preparatory work

| Package | Work grouped while the package is open | Boundary |
| --- | --- | --- |
| Vestigare | Iteration 1 model provenance plus optional child-Trace correlation fields needed by Iteration 5 | Do not create or display Replay child Traces until Iteration 5 |
| Repetere persistence/domain | Define the complete Experiment/Run foundation needed by Iterations 2–6 | Do not simulate later lifecycle transitions merely because their fields exist |
| Pontis | Add the narrow Replay execution-session establishment/registration and idempotent close contract for Iteration 3 | Avoid unrelated Pontis lifecycle redesign |
| Repetere runtime | Reuse one coherent Run lifecycle/cleanup abstraction across Iterations 3–6 | Enable behaviour incrementally and retain a test gate after each iteration |
| Praebere and Nuntius | Use the existing model discovery, selection/reservation, activation and release contracts | Change these packages only if integration proves a concrete contract gap |
| Repetere UI | Implement Iteration 7 against stable backend representations | Avoid UI-driven backend schema or lifecycle decisions |

## Iteration discipline

For each iteration:

1. confirm the exact package and configuration files to change;
2. implement only the accepted behaviour plus explicitly identified preparatory capacity;
3. run package-level automated tests;
4. provide a deployable package and any live configuration changes;
5. run the agreed focused live test;
6. record evidence, defects and decisions;
7. proceed only after the acceptance gate is agreed as satisfied.

This sequence supersedes the earlier broad milestone ordering only by making package-local preparatory work explicit. It does not combine the behavioural milestones or remove their independent test gates.
