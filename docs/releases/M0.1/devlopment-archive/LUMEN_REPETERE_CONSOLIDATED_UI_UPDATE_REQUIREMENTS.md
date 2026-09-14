# Lumen Repetere — Consolidated UI Update Requirements

**Status:** Authoritative Repetere UI implementation reference  
**Target:** M0.1 / Phase 9 UI completion  
**Updated:** 12 September 2026  
**Scope:** Repetere operator UI only

## 1. Purpose

This document consolidates the Repetere UI requirements previously spread across:

- `LUMEN_REPETERE_UI_UPDATE_REQUIREMENTS.md`
- `REPETERE_CHILD_OF_STAGED_REPLAY_UI_AND_STATE_CLEANUP`
- `LUMEN_REPETERE_MATCHED_AND_DIVERGENT_REPLAY_BEHAVIOUR`

Only requirements that affect the Repetere operator interface, visible state, hierarchy, refresh behaviour, actions, and evidence presentation are retained here. Service-internal orchestration and implementation behaviour are included only where needed to define what the UI must display.

This document also adds the requirement that every entry in **Trace recordings** displays the model used by that Trace.

---

## 2. UI Information Architecture

### 2.1 Experiment, Run and child Trace hierarchy

Repetere must present Replay work as a durable Experiment containing independent Runs and their child Traces.

```text
Source Trace
  -> Experiment
       -> Run 1
            -> Child Trace 1
       -> Run 2
            -> Child Trace 2
       -> Run 3
            -> No child Trace if execution never started
```

A Replay-created Trace must be displayed beneath or visibly associated with its Run. It must not appear to the operator as an unrelated top-level source Trace.

### 2.2 Experiment display

The Experiment view should expose, where applicable:

- Experiment name;
- source Trace;
- current actionable Experiment state;
- required model;
- schedule summary;
- Run history.

The Experiment's current actionable state must remain distinct from the result of its latest Run.

### 2.3 Run display

Each Run should expose, where applicable:

- attempt/run identity;
- lifecycle/current state;
- terminal result;
- requested/started/completed timing;
- Replay session;
- effective model;
- matched progress;
- first divergence evidence;
- failure stage/reason;
- cleanup/recovery state;
- associated child Trace;
- child Trace recording metadata.

Previous Runs and their evidence must remain visible after later Runs are created.

---

## 3. Status Presentation

### 3.1 Use coloured text rather than lifecycle pills

Ordinary Trace, Experiment and Run lifecycle/result states must be displayed as coloured text rather than pills or badges.

This applies to states such as:

- `Staged`
- `Preparing`
- `Running`
- `Recording`
- `Matched`
- `Diverged`
- `Failed / Incomplete`
- `Recovery Required`

Pills/badges should be reserved for cases where stronger treatment is useful, such as service-level `Available` indicators.

### 3.2 Simplify visible wording

Do not display redundant combinations such as:

```text
Staged for replay
Matched · Completed
Diverged · Completed
Failed · Completed
```

Use concise operator-facing result text:

```text
Staged
Running
Matched
Diverged
Failed / Incomplete
```

Internal lifecycle state may still distinguish execution completion from result, but the visible primary result should not append redundant `Completed` text.

### 3.3 Result and actionable state are separate

An Experiment is reusable and should not itself be labelled `Matched`, `Diverged`, or `Failed / Incomplete`.

Those are Run results.

After a clean terminal Run, the Experiment may return to `Staged` while the completed Run continues to show its own result.

---

## 4. Replay Start and Active-State Behaviour

### 4.1 Running is success-in-progress

A successfully accepted Replay that is still executing must not generate an operation-error banner.

Do not display:

```text
Operation failed: replay running
```

`Preparing`, `Accepted`, and `Running` are normal active states.

### 4.2 Immediate new-Run state

When **Run again** or another execution starts, the UI must immediately represent the new Run rather than leaving the previous Run's terminal state as the apparent current state.

The new Run should appear initially as:

```text
Preparing
```

and progress automatically through accepted/running state as authoritative backend state changes.

### 4.3 Clear transient stale UI

Starting another Run must clear transient UI left by the previous action, including:

- stale operation-error banners;
- stale success/completion banners;
- stale current-result text that could be mistaken for the new Run's state.

Historical Run evidence itself must not be cleared.

---

## 5. Automatic Refresh and Polling

### 5.1 Poll active Runs

While a Run is `Preparing`, `Accepted`, or `Running`, Repetere should use bounded polling consistent with the existing UI pattern.

Polling should refresh:

- Run lifecycle;
- Run result;
- matched progress;
- first divergence;
- child Trace recording state;
- child Trace message count;
- child Trace exchange count;
- child Trace duration;
- cleanup/recovery outcome.

### 5.2 Final authoritative refresh

When the Run reaches a terminal state, active polling should stop and Repetere must perform a final authoritative refresh of:

1. the Run;
2. the associated child Trace;
3. the Experiment's actionable state; and
4. the **Trace recordings** list.

No browser refresh or tab change should be required.

### 5.3 Trace recordings list refresh

When a Replay-created Vestigare recording ends, the **Trace recordings** list must refresh automatically.

The newly completed/updated Trace must immediately reflect its authoritative state and metadata.

---

## 6. Trace Recordings Display

Each Trace shown in **Trace recordings** should display, where available:

- recording/lifecycle state;
- **model used by the Trace**;
- message count;
- exchange count;
- duration;
- completion state/metadata.

### 6.1 Model display — new requirement

Every Trace entry in **Trace recordings** must display the model associated with that Trace.

The displayed model must come from the Trace's recorded model provenance rather than the currently selected model in Praebere or another session's model.

For example:

```text
Trace recording
Model: qwen2.5-coder:14b
Messages: 13
Exchanges: 6
Duration: 00:01:42
```

If authoritative model provenance is unavailable, the UI should display an explicit neutral value such as:

```text
Model: Unknown
```

rather than infer or silently substitute a model name.

### 6.2 No stale Trace metadata

A completed Trace must not remain displayed as:

```text
Recording
Messages: 0
Exchanges: —
Duration: —
```

after Vestigare has authoritative completed metadata.

---

## 7. Matched Replay Presentation

A fully matched Replay should progress visually through:

```text
Staged
  ↓
Preparing / Running
  ↓
Matched
```

Once matched:

- the Run displays `Matched`;
- its child Trace is refreshed to completed state;
- the Experiment returns to its appropriate actionable state, normally `Staged`;
- the Trace recordings list refreshes.

The operator should not need to know or see an additional `Completed` suffix to understand the result.

---

## 8. Divergent Replay Presentation

### 8.1 Divergence is a result, not an infrastructure error

A behavioural divergence must not be presented through the generic operation-error banner.

It is a valid Replay result.

### 8.2 First divergence evidence

Where available, the Run display should show:

- number of matched steps;
- first divergent/fork step;
- expected behaviour;
- observed behaviour.

For example:

```text
Divergence detected

Matched steps: 2
Fork at step 3

Expected:
<recorded behaviour>

Observed:
<live behaviour>
```

### 8.3 Divergence detection is not terminal completion

When the first behavioural mismatch is detected, the Run remains active while the divergent interaction continues.

The UI must not prematurely present the Run as terminal merely because the first fork has been found.

It should continue to show an active state while retaining the divergence indication/evidence.

Only after the complete divergent interaction has ended should the terminal Run result be displayed as:

```text
Diverged
```

### 8.4 Final divergent refresh

At terminal divergence:

- the Run result becomes `Diverged`;
- the full child Trace is refreshed;
- the Trace recordings list is refreshed;
- the Experiment returns to its appropriate actionable state if cleanup is complete.

---

## 9. Failed / Incomplete Presentation

Infrastructure or prerequisite failures that prevent a complete comparable Replay should be presented as:

```text
Failed / Incomplete
```

rather than `Diverged` or an ambiguous `Failed · Completed`.

Where available, display a concise reason such as:

- required model unavailable;
- different model currently selected/locked;
- Vestigare recording conflict;
- session creation failure;
- model activation failure;
- runtime/infrastructure failure;
- timeout or cancellation.

A failed Run remains evidence and must remain visible in Run history.

---

## 10. Error Banner Rules

Generic operation-error banners are reserved for failed user operations or infrastructure requests, for example:

```text
Unable to create Experiment
Unable to start Replay
Unable to contact required service
Unable to retry cleanup
Unable to unstage Experiment
```

Do not use the generic operation-error banner for:

- `Preparing`;
- `Accepted`;
- `Running`;
- `Matched`;
- `Diverged`;
- a normal terminal Replay result.

Behavioural evidence belongs in the Run result/evidence display.

---

## 11. Run Again

**Run again** must:

- clear transient banners from the previous action;
- retain every earlier Run;
- retain every earlier child Trace;
- immediately create/show the new Run in `Preparing`;
- show progress automatically;
- associate the new child Trace when recording begins;
- display the authoritative terminal result;
- refresh final child Trace metadata;
- refresh the Trace recordings list.

A retry is always visually represented as a new Run. It must not overwrite or mutate an earlier Run into the new outcome.

---

## 12. Recovery UI

When cleanup cannot be confirmed, the UI should distinguish this from an ordinary failed Run.

The Experiment should expose a recovery state such as:

```text
Recovery Required
```

In that state:

- **Run again** is disabled until safe cleanup is confirmed;
- a guarded **Retry Cleanup / Recover** action is available;
- **Unstage Experiment** remains separately available where permitted;
- the failed Run and all evidence remain visible.

Recovery may update cleanup status, but it must not rewrite the original Run result.

---

## 13. Action Availability

After a clean terminal Run, the Experiment should return to an actionable `Staged` state and make the appropriate actions available, including:

- **Run again**
- **Unstage Experiment**

When recovery is required, action availability must reflect that state rather than allowing another unsafe Run.

---

## 14. Scheduling Information

Where an Experiment is managed by Fiducia, the Repetere UI should show an appropriate schedule summary.

Scheduled execution acceptance and terminal Run result must remain visually distinct.

A scheduled trigger being accepted does not mean the resulting Replay has matched or completed successfully.

---

## 15. No Manual Refresh Requirement

The normal Replay workflow must not require the operator to:

- refresh the browser;
- change tabs;
- leave and re-enter Repetere;
- manually refresh Trace recordings.

The UI must automatically converge on authoritative backend state from Replay start through terminal Run and child Trace completion.

---

## 16. Intended Operator Flows

### 16.1 Matched

```text
Experiment: Staged
       ↓
Run: Preparing
       ↓
Run: Running
       ↓
Run: Matched
       ↓
Child Trace refreshed
Trace recordings refreshed
Experiment: Staged
```

### 16.2 Diverged

```text
Experiment: Staged
       ↓
Run: Preparing
       ↓
Run: Running
       ↓
First divergence detected
Run remains active
       ↓
Live continuation completes
       ↓
Run: Diverged
       ↓
Child Trace refreshed
Trace recordings refreshed
Experiment: Staged
```

### 16.3 Failed prerequisite

```text
Experiment: Staged
       ↓
Run: Preparing
       ↓
Run: Failed / Incomplete
Reason displayed
       ↓
No misleading child Trace if recording never began
Experiment: Staged
```

### 16.4 Cleanup failure

```text
Run ends
       ↓
Cleanup unconfirmed
       ↓
Experiment: Recovery Required
       ↓
Run again disabled
Retry Cleanup / Recover available
```

---


## 17. Trace Inspection Navigation and Consistency

### 17.1 Inspection navigation

When the operator enters the Trace recording inspector from Repetere, the inspection view must provide an obvious navigation path back to the Repetere operator screen.

The preferred presentation is a breadcrumb such as:

```text
Repetere > Trace recordings > <recording name>
```

where **Repetere** or **Trace recordings** is clickable.

A clearly labelled control such as:

```text
← Back to Trace recordings
```

is also acceptable.

The operator must not be required to know that clicking the top-level Repetere Servire tab is the only way to return to the main Repetere view.

### 17.2 Uniform inspection of source and Replay-created Traces

The **Inspect** action must use the same Trace inspection presentation for:

- client/source Trace recordings; and
- Replay-created child Trace recordings.

A valid Replay-created child Trace must therefore open the normal Trace inspection experience, including the same recording overview, traffic classification, semantic conversation, search/conversation evidence, and related inspection controls available for a client/source Trace.

A Replay-created child Trace must not fall through to a raw API/JSON error response merely because its exchange structure differs from a client/source recording.

Where a recording is genuinely incomplete or malformed, Repetere should still render the inspection UI shell and present the incomplete or malformed state as an operator-facing condition. It should not replace the inspection screen with raw API output such as:

```json
{"detail":"recording contains an incomplete or malformed exchange"}
```

The inspection UI should identify the affected exchange or incomplete evidence where possible while preserving access to the remainder of the recording.

---

## 18. UI Acceptance Criteria

The Repetere UI update is complete when:

1. Experiment, Run and child Trace hierarchy is clear.
2. Previous Runs and their evidence remain inspectable.
3. Run result and Experiment actionable state are visually distinct.
4. Ordinary lifecycle/result states use coloured text rather than pills.
5. `Staged for replay` is reduced to `Staged`.
6. Visible Run results do not append redundant `· Completed`.
7. `Running` is never shown as an operation failure.
8. A newly started Run immediately replaces stale current-state presentation.
9. `Run again` clears transient banners but preserves historical evidence.
10. Active Runs refresh automatically.
11. Polling covers Run state, progress, divergence, child Trace metadata and cleanup state.
12. Polling stops at terminal state.
13. A final authoritative refresh occurs at terminal state.
14. The Experiment actionable state refreshes after the Run ends.
15. The **Trace recordings** list refreshes when Replay recording ends.
16. Trace recording state, message count, exchange count and duration refresh automatically.
17. **Every Trace entry displays its recorded model.**
18. Missing model provenance is displayed explicitly rather than inferred.
19. Replay-created child Traces are visibly associated with their Runs.
20. Matched Replay is shown as `Matched`.
21. Behavioural divergence is shown as `Diverged`, not generic failure.
22. First-divergence evidence is displayed where available.
23. A divergent Run remains active until the complete continuation ends.
24. Infrastructure/prerequisite failures are shown as `Failed / Incomplete` with a reason.
25. Failed Runs remain preserved as evidence.
26. Generic error banners are reserved for actual failed operations.
27. Recovery-required state disables unsafe rerun and exposes recovery controls.
28. Schedule information is visible where Fiducia manages the Experiment.
29. Asynchronous acceptance is not presented as terminal success.
30. No normal Replay lifecycle requires manual browser refresh or tab switching.
31. The Trace inspector provides an obvious breadcrumb or back control to return to Repetere / Trace recordings.
32. Client/source Traces and Replay-created child Traces use the same inspection experience; malformed/incomplete recordings are presented within the inspector rather than as raw API/JSON output.

---

## 19. Implementation Boundary

This document deliberately defines **Repetere UI requirements only**.

Detailed service ownership, model reservation, Pontis/ACP routing, Vestigare recording internals, Praebere lifecycle, Fiducia PID handling, Servire managed-process behaviour, and Replay comparison algorithms remain governed by their respective implementation specifications.

Those behaviours are referenced here only when they determine what Repetere must present to the operator.
