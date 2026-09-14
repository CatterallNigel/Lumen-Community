# Lumen Repetere — UI Update Requirements

**Status:** Consolidated implementation reference  
**Scope:** Repetere operator UI / Replay lifecycle / Trace refresh behaviour  
**Updated:** 11 September 2026

## 1. Purpose

This document consolidates the agreed Repetere UI changes for the M0.1 Replay workflow.

It brings together the earlier child-of-staged UI/state-cleanup requirements, the later definitive matched/divergent Replay behaviour, and the subsequent UI refinements agreed during testing.

The primary goals are:

- remove unnecessary lifecycle/status pills;
- simplify status wording;
- ensure Replay and Trace state refresh automatically;
- distinguish behavioural divergence from infrastructure failure;
- keep Replay-created Trace information authoritative and current;
- ensure `Run again` starts from clean UI state;
- avoid requiring any manual browser or tab refresh during normal operation.

---

## 2. Status Presentation

### 2.1 Remove Replay/Trace lifecycle status pills

Replay and Trace lifecycle/result states must no longer be shown inside pills or badges.

Examples include:

- `Completed`
- `Staged for replay`
- `Recording`
- `Running`
- `Matched`
- `Diverged`
- `Failed`

These states should instead be rendered as ordinary coloured text.

Colour provides sufficient visual distinction.

Pills/badges should remain reserved for cases where the stronger visual treatment is useful, such as service-level indicators like `Available`.

### 2.2 Simplify lifecycle wording

Remove redundant lifecycle wording where the UI context already makes the state obvious.

In particular:

- `Staged for replay` becomes `Staged`
- `Matched · Completed` becomes `Matched`
- `Diverged · Completed` becomes `Diverged`
- `Failed · Completed` becomes `Failed`

The Replay result is the important information. Completion of the Replay engine should not obscure or duplicate the experiment result.

Expected visible lifecycle:

```text
Staged
  ↓
Running
  ↓
Matched
```

or:

```text
Staged
  ↓
Running
  ↓
Diverged
```

Status text should use colour rather than badge/pill styling.

---

## 3. Running State

### 3.1 `Running` is not an error

A Replay accepted asynchronously and currently executing is a valid successful state.

The UI must not display messages such as:

```text
Operation failed: replay running
```

or any equivalent error banner merely because the Replay remains active.

### 3.2 Immediate transition on Replay start

Once Replay start has been accepted successfully, the UI should immediately show the new Run as:

```text
Running
```

The previous Run's visible state must not remain displayed as the active state.

---

## 4. Automatic Replay Refresh

### 4.1 Poll while a Replay is active

While a Replay is running, Repetere should automatically refresh the authoritative execution state.

The refresh should update, where applicable:

- Replay status;
- Run status;
- matched-step count;
- divergence/result state;
- first-fork information;
- child Trace status;
- child Trace metadata.

The operator must not need to refresh the browser or change tabs to obtain current state.

### 4.2 Stop polling at terminal state

Polling should stop once the Run reaches a terminal state.

Examples include:

- `Matched`
- `Diverged`
- `Failed`
- `Terminated`
- `Cancelled`
- `Timed out`

### 4.3 Final authoritative refresh

When a terminal state is reached, Repetere must perform one final refresh to obtain the authoritative final state for:

1. the Replay Run; and
2. the associated Vestigare child Trace.

---

## 5. Trace Recordings Refresh

### 5.1 Refresh the Trace recordings list when Replay recording ends

When Vestigare completes the Replay-created recording, Repetere must refresh the **Trace recordings** list.

The completed Replay Trace must appear immediately with its authoritative state.

The operator must not need to:

- refresh the browser;
- change tabs;
- leave and re-enter the Repetere view.

### 5.2 Refresh all Trace metadata

The completed Trace entry must refresh, where available:

- recording/completion state;
- message count;
- exchange count;
- duration;
- completion metadata.

The UI must not remain in a stale state such as:

```text
Recording
Messages: 0
Exchanges: —
Duration: —
```

after Vestigare has already completed the recording.

---

## 6. Replay Trace Relationship

### 6.1 Replay-created Trace belongs to the Replay Run

A Replay-created Vestigare Trace should be visibly associated with the relevant Replay Run rather than presented as though it were an unrelated recording.

The conceptual structure is:

```text
Staged Replay Experiment
    |
    +-- Replay Run 1
    |     |
    |     +-- Replay Trace
    |
    +-- Replay Run 2
          |
          +-- Replay Trace
```

### 6.2 Preserve Run history

Multiple Runs may exist beneath the same staged Replay Experiment.

Each Run should retain:

- its result;
- matched-step information;
- divergence evidence;
- associated child Trace;
- failure evidence where applicable.

A new Run must not overwrite previous Run evidence.

---

## 7. `Run again` Behaviour

Starting another Replay Run must:

- clear stale error banners from the previous Run;
- clear stale completion/result text from the previous Run;
- immediately show the new Run as `Running`;
- create and associate a new Replay Trace;
- begin updating progress automatically;
- display the terminal result automatically;
- retain previous Run evidence.

The new Run owns the current active UI state.

The previous Replay's terminal message must not remain visible as though it belongs to the new execution.

---

## 8. Behavioural Divergence vs Infrastructure Failure

The UI must distinguish clearly between:

- Replay could not be started;
- Replay infrastructure/runtime failure;
- Replay completed and matched;
- Replay completed with behavioural divergence.

A behavioural divergence is a valid experimental result.

It is not an infrastructure failure.

### 8.1 Divergence evidence

When a behavioural fork occurs, the result display should expose the relevant evidence, including where available:

- matched steps;
- fork step;
- expected behaviour;
- observed behaviour.

For example:

```text
Diverged

Matched steps: 1
Fork at step 2

Expected:
<recorded behaviour>

Observed:
<live behaviour>
```

### 8.2 Error banners

Generic operation-error banners should only be used for actual operation failures.

Examples:

```text
Unable to contact Nuntius
Vestigare rejected recording start
Moderari prerequisite failed
Replay execution could not be started
```

The following are not operation failures:

```text
Replay accepted and running
Replay completed and matched
Replay completed with behavioural divergence
```

Behavioural divergence belongs in the Replay result/evidence display.

---

## 9. Divergent Replay Lifecycle

A first behavioural mismatch does not mean that the Replay has completed.

When divergence is detected:

1. Repetere records the first-fork evidence.
2. The Run remains active.
3. Repetere becomes transparent for the live continuation.
4. Tool execution and the final answer may continue.
5. Vestigare continues recording.
6. Pontis eventually reports that the complete interaction has ended.
7. Repetere then completes the Run and child Trace.

The UI must therefore not change to a terminal `Diverged` result immediately when the first mismatch occurs if the conversation is still continuing.

The Run should remain visibly active until the complete divergent interaction has ended.

Only after the complete continuation has finished should the UI show the terminal:

```text
Diverged
```

---

## 10. Matched Replay Lifecycle

For a fully matching Replay, Repetere itself knows when every meaningful source exchange has been reproduced and matched.

At that point Repetere can:

- mark the Run matched;
- stop the child Vestigare recording;
- complete the child Trace;
- close the Replay session;
- expose the terminal result.

The UI should then show:

```text
Matched
```

and perform the final Run and Trace refresh.

---

## 11. Request Acceptance Is Not Completion

Asynchronous acceptance of a Replay request must not be presented as Replay success.

The UI lifecycle should remain explicit:

```text
Staged
  ↓
Running
  ↓
Matched / Diverged / Failed / other terminal state
```

The staged Experiment display must continue updating until the authoritative terminal state is known.

---

## 12. Pre-execution Failures and Deferred States

If Replay cannot safely begin because of a prerequisite problem, the UI must present a meaningful explicit state.

Examples include:

- required model unavailable;
- runtime locked to a different model;
- Vestigare recording boundary unavailable;
- provider/runtime prerequisite failure;
- session creation failure.

The UI must not:

- silently substitute another model;
- wait indefinitely without explanation;
- present an empty Replay as successful;
- present a failed prerequisite as a behavioural divergence.

Where applicable, the operator-facing state should expose the reason for non-execution.

---

## 13. Failed/Incomplete Runs

A failed or incomplete Replay Run must preserve its evidence.

Retrying later must create a new Run rather than overwrite the failed one.

Previous evidence should remain available for inspection.

---

## 14. No Manual Refresh Requirement

Normal Repetere operation must not require a manual browser refresh or tab change.

From start to finish, the UI should automatically converge on authoritative backend state:

```text
Staged
  ↓
Running
  ↓
Matched / Diverged / Failed
  ↓
Replay Trace completed and refreshed
```

The associated Trace recording must also visibly move from active recording state to its authoritative completed state automatically.

---

## 15. UI Acceptance Criteria

The Repetere UI update is complete when all of the following are true:

1. Replay/Trace lifecycle statuses are shown as coloured text rather than pills.
2. Redundant wording such as `Staged for replay` and `· Completed` has been removed from visible state labels.
3. `Running` is never presented as an operation failure.
4. A Replay entering `Running` replaces stale state from the previous Run immediately.
5. Active Replay state updates automatically.
6. Polling ends at terminal state.
7. A final authoritative Replay refresh occurs at terminal state.
8. The associated Vestigare child Trace is refreshed at terminal state.
9. The overall **Trace recordings** list is refreshed when the Replay recording ends.
10. Trace metadata updates without requiring browser refresh.
11. Replay-created Traces are visibly associated with their Runs.
12. Multiple Runs retain their own evidence.
13. `Run again` clears stale banners and stale result text.
14. Behavioural divergence is presented as a result rather than an infrastructure error.
15. Divergence evidence is available to the operator.
16. A divergent Replay remains active until the complete live continuation ends.
17. Matched Replay completes cleanly and refreshes its child Trace.
18. Failed or deferred prerequisites are shown explicitly.
19. Failed Run evidence is retained.
20. No normal Replay lifecycle requires manual page or tab refresh.

---

## 16. Intended Operator Experience

The intended normal matched flow is:

```text
Staged
  ↓
Running
  ↓
Matched
```

The intended divergent flow is:

```text
Staged
  ↓
Running
  ↓
Divergence detected
  ↓
Live continuation remains active
  ↓
Diverged
  ↓
Fork evidence available
```

In both cases, the associated Vestigare Trace should automatically move to its authoritative completed state and the **Trace recordings** list should refresh accordingly.
