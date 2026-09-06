# Repetere Experiment, Run, Child-Trace UI and State Cleanup

**Status:** Required M0.1 Phase 9 work  
**Target:** Phase 9 — Experiment Structure  
**Originally recorded:** 2026-08-28  
**Last reconciled:** 2026-09-06  
**Scope:** Repetere, Vestigare and Fiducia integration; Repetere operator UI; Fiducia/Servire lifecycle corrections

## Revision History

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-08-28 | Initial child-of-staged Replay UI and cleanup requirements. |
| 1.1 | 2026-08-31 | Added Pontis session identity, model provenance, concurrency, Vestigare arbitration and Fiducia scheduling requirements. |
| 2.0 | 2026-09-06 | Recast staged Replay as an explicit Experiment with immutable runs; reconciled completed N9.6.3, deferred N9.6.4, fail-fast model policy and current Phase 9 scope. |

## 1. Purpose

Phase 9 introduces Experiment as the durable parent concept for controlled repeated
execution. This document defines the associated Repetere state model, child Trace
relationship, UI behaviour, failed-run recovery and Fiducia coordination.

It supersedes the older implication that a single staged Replay record can represent
both a reusable experiment and the mutable state/result of its latest execution.

## 2. Established Baseline

The current asynchronous Replay start boundary is retained:

1. Repetere establishes all execution prerequisites.
2. Moderari confirms session-scoped Pass-through through Nuntius.
3. Vestigare confirms that recording has been started.
4. Repetere starts the Replay asynchronously.
5. The caller receives accepted/running state without waiting for completion.

Repetere must not wait for a later Vestigare status event after receiving the
authoritative successful start-recording response. Asynchronous acceptance is not the
terminal Experiment result.

## 3. Canonical Domain Model

### 3.1 Experiment

An Experiment is a reusable definition for controlled repeated execution. For M0.1 it
contains at least:

- stable `experiment_id`;
- researcher-supplied name;
- source Vestigare `trace_id`;
- required provider/model derived from that Trace;
- created/updated timestamps;
- current actionable state;
- ordered run identifiers; and
- scheduling relationship where Fiducia is used.

An Experiment is not `MATCHED`, `DIVERGED` or `FAILED / INCOMPLETE`. Those are run
results. The Experiment remains available for another run until deliberately unstaged
or removed.

### 3.2 Experiment Run

Every execution attempt creates an immutable run record containing at least:

- stable `run_id` and parent `experiment_id`;
- attempt number;
- fresh Pontis Replay `session_id`;
- source `trace_id`;
- Replay-created child `trace_id`, when recording was started;
- required and effective provider/model identities;
- requested, started and completed timestamps;
- lifecycle state;
- terminal result;
- matched progress and first divergence where available;
- failure stage and concise reason; and
- cleanup/session-release outcome.

A rejected prerequisite still produces a failed run record. It may legitimately have
no child Trace because model execution and recording never began.

### 3.3 Child Trace

Every Replay-created Trace must reference its `experiment_id`, `run_id` and Replay
session. It is displayed beneath its run, not as an unrelated top-level source Trace.

```text
Source Trace
  -> Experiment
       -> Run 1 -> child Trace 1
       -> Run 2 -> child Trace 2
       -> Run 3 -> no child Trace (prerequisite failure)
```

## 4. State Model

### 4.1 Experiment Action State

Use an explicit Experiment action state independent of run outcome:

| State | Meaning |
|---|---|
| `STAGED` | Ready to create a new run |
| `RUNNING` | One current run is executing; M0.1 permits only one run at a time |
| `RECOVERY_REQUIRED` | The latest run could not complete cleanly or cleanup is unconfirmed; evidence is preserved and recovery actions are available |
| `UNSTAGED` | No longer available for execution but retained according to evidence policy |

After a clean terminal `MATCHED`, `DIVERGED` or `FAILED / INCOMPLETE` run, the
Experiment returns to `STAGED`. Use `RECOVERY_REQUIRED` only when the Experiment cannot
safely accept another run until cleanup/reset is resolved.

### 4.2 Run Lifecycle

| State | Meaning |
|---|---|
| `PREPARING` | Creating session and validating prerequisites |
| `ACCEPTED` | Prerequisites succeeded and asynchronous execution was accepted |
| `RUNNING` | Replay execution is active |
| `COMPLETED` | Execution ended and a terminal result was persisted |
| `CLEANUP_REQUIRED` | Execution ended but session/recording cleanup is incomplete or unconfirmed |

### 4.3 Terminal Run Result

| Result | Meaning |
|---|---|
| `MATCHED` | Comparable execution completed and matched source evidence |
| `DIVERGED` | Comparable execution completed with a behavioural difference |
| `FAILED / INCOMPLETE` | Prerequisites or infrastructure prevented a complete comparable result |

Do not present behavioural divergence as generic operation failure. Do not display the
ambiguous historical combination `FAILED · COMPLETED`; present `DIVERGED` when a
completed run contains a behavioural fork and `FAILED / INCOMPLETE` for incomplete
infrastructure execution.

## 5. Run Creation and Isolation

Every manual, repeated or Fiducia-scheduled attempt must:

1. allocate a new `run_id`;
2. establish a fresh Pontis Replay session;
3. propagate that session unchanged through Repetere, Praebere, Moderari and
   Vestigare;
4. create a new child Trace only after recording prerequisites succeed;
5. persist the terminal run independently; and
6. release only that run's session/reservation contribution.

Never reuse the source session, a console session, a previous run's session or another
active Replay session. Sequential retries must not inherit prior model context,
Moderari state or tool state.

## 6. Model and Recording Prerequisites

`M0.1_TRACE_MODEL_REPLAY_REQUIREMENTS` is authoritative for detailed model rules.
In summary:

- read provider/model from the source recording at actual run start;
- use Praebere's cached model catalogue and normal selection/reservation operation;
- continue when no model is selected or the selected model is an exact match;
- fail immediately when another model is selected, reserved or locked;
- do not substitute a model or wait indefinitely;
- allow Praebere to test provider reality during lazy activation;
- cancel the run's failed reservation if activation fails; and
- do not start execution without its Vestigare recording boundary.

N9.6.4 provider-neutral readiness is deferred and is not an M0.1 prerequisite.

M0.1 permits concurrent console and Replay execution only when both sessions have
explicitly reserved the same runtime-global model. Context, responses, tools and Trace
evidence remain isolated by Pontis session identity. Completion of one session must not
release another session's reservation or execution state.

Vestigare permits one active recording. If recording is already owned by another
session when a run starts, create a `FAILED / INCOMPLETE` run with a recording-conflict
reason. M0.1 does not introduce an indefinite hidden wait queue.

## 7. Failed-Run Recovery

A failed run is evidence and must never be overwritten, deleted or rewritten into a
later success.

After a cleanly contained failure:

- persist the run as `FAILED / INCOMPLETE`;
- persist its failure stage, reason and any available child Trace;
- close/deactivate its Pontis session and release its reservation idempotently;
- return the Experiment to `STAGED`; and
- offer **Run again** and **Unstage Experiment**.

If session release, recording stop or another cleanup boundary is unconfirmed:

- preserve the run in `CLEANUP_REQUIRED`;
- set the Experiment to `RECOVERY_REQUIRED`;
- disable **Run again** until safe cleanup is confirmed;
- offer a guarded **Retry Cleanup / Recover** action; and
- retain a separately authorised **Unstage** action that does not erase evidence.

Recovery updates the cleanup status of the failed run but must not change its terminal
result. A new execution is always a new run/session/Trace.

## 8. Repetere UI

### 8.1 Hierarchy

The UI must present:

```text
Source Trace
Experiment
  Latest/current action state
  Required model
  Schedule summary
  Run history
    Run result and timing
    Replay session
    Effective model
    First divergence or failure reason
    Child Trace and recording metadata
```

The researcher must be able to distinguish the terminal result of each run from the
current actionable state of the Experiment.

### 8.2 Refresh

While a run is preparing, accepted or running, use bounded polling consistent with the
existing UI pattern to refresh:

- run lifecycle and result;
- matched progress and first divergence;
- child Trace recording state, message/exchange counts and duration; and
- cleanup outcome.

Perform a final authoritative refresh at terminal state. No browser refresh should be
required. Routine UI polling remains outside the normal Servire Operational Log,
model context and Vestigare conversational Trace.

### 8.3 Presentation Rules

- `RUNNING` is success-in-progress, not an operation-error banner.
- `DIVERGED` is a behavioural result, not infrastructure failure.
- Generic error banners are reserved for failed user operations such as inability to
  create/retry/recover/unstage an Experiment.
- Ordinary Trace, Experiment and run states use coloured text rather than pills.
- Service-level availability may retain the stronger badge treatment.
- First divergence displays expected and observed evidence where available.
- Repetere divergence detection remains distinct from future Aestimare assessment.

### 8.4 Run Again

**Run again** must:

- clear only transient banners from the previous action;
- retain every earlier run and child Trace;
- immediately add a new run in `PREPARING`;
- allocate a fresh Pontis session;
- show current progress automatically; and
- display the authoritative terminal result and child Trace metadata.

## 9. Fiducia Coordination and Managed Lifecycle

Fiducia coordinates repeated runs against `experiment_id`. Each trigger creates one
new run through Repetere and records acceptance separately from the terminal result.
It must surface `MATCHED`, `DIVERGED` or `FAILED / INCOMPLETE`, plus the linked run and
child Trace.

The Phase 9 Fiducia/Servire pass must also correct the managed-process boundary:

### 9.1 Stale PID Recovery

On startup Fiducia must:

1. read an existing PID file;
2. verify that the PID identifies a live Fiducia process, not merely any live process;
3. reject duplicate startup only for a genuine running Fiducia instance;
4. remove/replace a stale or mismatched PID file;
5. log the recovery; and
6. continue startup.

Current live behaviour reports stale-PID recovery on every managed start. Investigate
the Servire stop/start ordering, Fiducia shutdown cleanup and PID write/remove race as
one lifecycle defect rather than weakening duplicate-process protection.

### 9.2 Managed Log Cleanup

Servire currently invokes Fiducia `clear-logs`, but the Fiducia CLI rejects the
argument with exit code 2. Align the managed command contract so **Clear Logs** succeeds
without starting another Fiducia instance or disturbing its PID state.

### 9.3 Runtime Files

PID, lock and transient runtime files belong in container-runtime storage. They must
not be packaged or mounted as durable application data in the M0.1 distribution.
Experiment, schedule and run evidence remains durable application data.

## 10. Minimum Acceptance

### Experiment and run

1. An Experiment has a stable identity and references one source Trace.
2. Each manual and scheduled attempt creates a new run.
3. Every run receives a fresh Pontis session.
4. Each created child Trace references its Experiment, run and Replay session.
5. Prior matched, diverged and failed evidence remains unchanged after **Run again**.
6. A clean failure returns the Experiment to an actionable `STAGED` state.
7. Unconfirmed cleanup produces `RECOVERY_REQUIRED` and blocks another run until
   guarded recovery succeeds.

### Execution prerequisites

8. Missing model provenance, unavailable model and different active model each fail
   before execution with a specific reason.
9. Same-model selection/reservation proceeds and remains isolated from console state.
10. Lazy activation failure cancels only the Replay reservation.
11. Vestigare contention creates a visible failed run; execution does not proceed
    without recording.
12. All terminal paths release only the Replay session and are idempotent.

### UI

13. Accepted/running state is not shown as operation failure.
14. The active run and child Trace refresh automatically through terminal state.
15. Run results and Experiment action state are displayed separately.
16. First divergence and child Trace metadata appear where available.
17. Lifecycle/result states use coloured text rather than ordinary pills.

### Fiducia/Servire

18. Fiducia coordinates repeated runs by Experiment identity and records terminal
    outcomes.
19. A genuine running Fiducia blocks duplicate startup.
20. A stale/mismatched PID is removed once, logged and does not recur on every clean
    managed restart.
21. Servire **Clear Logs** succeeds for Fiducia.
22. PID/runtime files are absent from durable container data.

### Quality

23. Repetere, Vestigare, Fiducia, Pontis, Praebere, Servire and any other modified
    component tests pass.
24. Ruff and mypy pass for every modified Python component.
25. Live manual and Fiducia-scheduled runs demonstrate `MATCHED`, `DIVERGED` and
    `FAILED / INCOMPLETE` without manual browser refresh.

## 11. Exit Condition

Phase 9 is complete when a researcher can create an Experiment from a source Trace,
run it repeatedly manually or through Fiducia, and inspect every attempt as an
independent run with its own session, child Trace, model condition, result and
divergence/failure evidence.

Failed runs remain preserved, the Experiment remains recoverable and reusable, and no
run can silently substitute a model, execute without recording, reuse another run's
session or leave ambiguous state that prevents deliberate recovery.
