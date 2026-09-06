# Repetere --- Child-of-Staged Replay UI and State Cleanup

**Status:** Planned\
**Target:** M0.1 --- Repetere child-of-staged work\
**Recorded:** 2026-08-28\
**Scope:** Repetere / Vestigare integration and Repetere operator UI

------------------------------------------------------------------------

## Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-08-28 | Nigel Catterall | Initial child-of-staged Replay UI and state-cleanup requirements. |
| 1.1 | 2026-08-31 | Nigel Catterall | Added requirements for Pontis-authoritative Replay session identity, source-model provenance, Praebere discovery/readiness/selection prerequisites, unavailable or locked model handling, concurrent console/Replay execution, Vestigare single-recording arbitration, Fiducia scheduling outcomes and cross-service acceptance. Recorded N9.5 concurrent-session/single-model evidence as the baseline rather than as Replay validation. |

------------------------------------------------------------------------

## 1. Context

During implementation and live testing of the N7+ Replay pass-through
prerequisite, Repetere was changed so that starting a Replay:

1.  requests Moderari pass-through through Nuntius;
2.  requests Vestigare to begin the Replay Trace;
3.  accepts Vestigare's successful `200 OK` response as sufficient
    confirmation that recording has been initiated;
4.  starts the Replay asynchronously; and
5.  returns the accepted/running state without waiting for the complete
    Replay execution.

This corrected the previous timeout behaviour.

Live testing confirmed that the underlying control path now operates
correctly.

The testing also exposed several Repetere UI/state-management issues.
These are not blockers for the N7+ prerequisite implementation and
should be addressed together with the planned **Replay Trace as
child-of-staged experiment** work.

------------------------------------------------------------------------

## 2. Architectural Result Already Established

The successful Replay start sequence is:

``` text
Operator / Client
      |
      v
   Repetere
      |
      +----> Nuntius ----> Moderari
      |                   set pass-through
      |                         |
      |                       success
      |
      +----> Nuntius ----> Vestigare
                          start recording
                                |
                              200 OK
                                |
                                v
                     recording initiated
      |
      v
Replay execution begins asynchronously
      |
      v
Repetere returns accepted/running state
```

Repetere MUST NOT wait for Vestigare to subsequently report that the
recording has entered the `recording` state.

A successful response from the Vestigare start-recording control
operation is the synchronization boundary.

------------------------------------------------------------------------

## 3. Observed UI Problems

### 3.1 `running` is incorrectly presented as an operation failure

After selecting **Run again**, Repetere can display:

``` text
Operation failed: replay running.
```

This is incorrect.

`running` is now a valid successful state following asynchronous Replay
acceptance.

The UI must distinguish between:

-   failure to start the Replay; and
-   successful Replay start with execution still running.

Expected behaviour:

``` text
Replay accepted
      ↓
Running
      ↓
Matched / Failed / other terminal result
```

There should be no error banner merely because execution remains in
progress.

### 3.2 Replay experiment state is not automatically refreshed

While the Replay executes in the background, the staged Replay
experiment card can remain in an earlier state.

For example:

``` text
RUNNING · RUNNING
```

can remain visible after the Replay has already completed.

Refreshing the Repetere page/tab causes the correct terminal state to
appear.

Therefore the authoritative backend state exists, but the Repetere UI is
not refreshing it appropriately.

The staged experiment display must update automatically while a Replay
is active and perform a final refresh when execution reaches a terminal
state.

### 3.3 Replay-created Trace information is stale

The Trace recording created automatically for Replay can appear in the
Repetere Trace list as:

``` text
recording
Messages: 0
Exchanges: —
Duration: —
```

even after Vestigare has completed the recording.

Vestigare itself correctly contains the completed recording and its
messages.

Repetere therefore needs to refresh the corresponding Trace metadata
during and following Replay execution.

On completion, the Repetere view should reflect the authoritative
Vestigare state, including where available:

-   recording status;
-   message count;
-   exchange count;
-   duration;
-   completion state.

A browser refresh must not be required to obtain these values.

------------------------------------------------------------------------

## 4. Behavioural Failure Must Remain Distinct from Infrastructure Failure

The live test also demonstrated an important distinction.

The Replay infrastructure can operate successfully while the Replay
experiment itself fails behaviourally.

Example observed during testing:

Original execution expected:

``` text
bash("echo '12 * 9 / 3' | bc")
```

During Replay, the model produced no corresponding tool call.

That is a legitimate behavioural divergence.

It should therefore be represented approximately as:

``` text
Replay accepted
      ↓
Running
      ↓
Failed — behavioural fork

Expected:
bash(...)

Observed:
no tool call
```

It must **not** be represented as failure of the Replay infrastructure
itself.

The UI should distinguish clearly between:

-   Replay could not be started;
-   Replay infrastructure/runtime failure;
-   Replay completed and matched;
-   Replay completed with behavioural divergence.

------------------------------------------------------------------------

## 5. Completion State Presentation

A completed Replay that has detected divergence may currently display
combinations such as:

``` text
FAILED · COMPLETED
Replay run completed.
```

This is technically understandable but visually and semantically
awkward.

The experiment has:

-   **completed execution**, and
-   **failed to reproduce the expected behaviour**.

The UI should make the behavioural result primary.

For example:

``` text
Failed · Completed

Behavioural fork detected.

Expected:
bash(...)

Observed:
No tool call
```

Similarly, a successful Replay may show:

``` text
Matched · Completed
```

The precise wording can be refined during implementation, but completion
of the Replay engine must not obscure the experiment result.

------------------------------------------------------------------------

## 6. Remove Status Pills from Replay/Trace States

The current pill/badge treatment for ordinary states such as:

``` text
recording
staged for replay
running
failed · completed
matched · completed
```

looks visually inappropriate in the Repetere workflow.

These states are informational rather than actions or exceptional
badges.

### UI rule

**Replay and Trace lifecycle/result status should be represented as
coloured text rather than pills.**

Examples:

``` text
Recording
Staged
Running
Matched · Completed
Failed · Completed
```

Colour already provides sufficient visual distinction.

Pills/badges should be reserved for states where the stronger visual
treatment is useful, such as the service-level `Available` indicator.

This change should be applied consistently when the child-of-staged UI
is implemented.

------------------------------------------------------------------------

## 7. Replay Trace Should Become a Child of the Staged Experiment

The automatically generated Replay recording currently appears as
another independent top-level Trace recording.

Conceptually, however, that Trace exists because of a particular staged
Replay experiment.

The intended relationship is:

``` text
Original Trace
      |
      v
Staged Replay Experiment
      |
      +---- Replay Run 1
      |       |
      |       +---- Replay Trace
      |
      +---- Replay Run 2
      |       |
      |       +---- Replay Trace
      |
      +---- ...
```

The Replay-created Trace should therefore be represented as a **child of
the staged experiment/run**, rather than appearing to be an unrelated
recording.

This relationship will also provide a natural location for displaying:

-   current Replay state;
-   Replay result;
-   matched steps;
-   first behavioural fork;
-   associated Vestigare Trace;
-   subsequent `Run again` executions.

------------------------------------------------------------------------

## 8. Refresh Behaviour

When implementing child-of-staged, Repetere should have explicit refresh
behaviour for active experiments.

While a Replay is running:

``` text
poll / refresh experiment state
        |
        +---- Replay status
        +---- run status
        +---- matched steps
        +---- divergence/result
        +---- child Trace status
        +---- child Trace metadata
```

Polling should stop when the run reaches a terminal state.

A final refresh should then obtain the authoritative terminal state for
both:

1.  the Replay run; and
2.  its Vestigare child Trace.

The operator should not need to refresh the browser manually.

------------------------------------------------------------------------

## 9. `Run again` Behaviour

`Run again` must transition the existing staged experiment cleanly into
another run.

Expected sequence:

``` text
Completed experiment
      |
   Run again
      |
      v
New Replay run
      |
      +---- new child Trace
      |
      v
Running
      |
      v
Terminal result
```

Starting another run must:

-   clear stale error banners from the previous run;
-   immediately show the new running state;
-   create/associate the new Replay Trace;
-   update progress while execution continues;
-   automatically display the terminal result;
-   retain previous run evidence rather than overwriting it where the
    child-of-staged model requires historical runs.

------------------------------------------------------------------------

## 10. Error Banner Behaviour

The general operation-error banner must only be used for actual
operation failures.

Examples include:

``` text
Unable to contact Nuntius
Vestigare rejected recording start
Moderari prerequisite failed
Replay execution could not be started
```

The following are **not** operation failures:

``` text
Replay accepted and running
Replay completed with behavioural divergence
Replay completed and matched
```

A behavioural divergence belongs to the Replay result/evidence display
rather than the generic operation-error banner.

------------------------------------------------------------------------

## 11. Replay Session, Model and Concurrency Requirements

The N9.5 provider/model work changes assumptions that existed when the original
Replay path was implemented. Moderari previously created a session identifier when
one was absent. Pontis is now authoritative for Lumen session identity, while
Praebere is authoritative for the runtime-global selected model and its execution
lock.

The following requirements are therefore part of the child-of-staged Replay work and
must be verified before M0.1 can claim scheduled Replay is safe alongside ordinary
console use.

### 11.1 N9.5 Baseline Already Demonstrated

Live N9.5 testing on 2026-08-31 demonstrated the ordinary-client baseline:

- an external HTTP session selected `qwen2.5-coder:14b` through
  Pontis → Nuntius → Praebere;
- Pontis activated that session as an execution session before its first model
  interaction;
- Praebere locked the runtime-global selection after execution began;
- a subsequently established Rogare session observed the same selected and loaded
  model;
- an attempted change to `qwen2.5-coder:14b-32k` returned `409 Conflict`;
- the authoritative selection remained `qwen2.5-coder:14b` with one active execution
  session.

This proves concurrent ordinary sessions share one locked runtime-global model. It
does **not** yet prove Replay session identity, Replay model acquisition, concurrent
ordinary/Replay execution or scheduled-Replay failure handling.

### 11.2 Pontis-Authoritative Replay Session Identity

Every Replay run must have a fresh authoritative Pontis `session_id`. The Replay run,
its Moderari policy prerequisite, its Praebere control requests, its model execution
and its Vestigare child Trace must all use that same identity.

Repetere must not depend on Moderari creating a missing session identifier. It must
not reuse:

- the source Trace's live session identifier;
- the parent staged Experiment's previous Replay session identifier;
- another Replay run's session identifier; or
- an active Rogare/Pi/external-client session identifier.

The required relationship is:

``` text
staged_experiment_id
    -> replay_run_id
        -> fresh Pontis replay_session_id
            -> Praebere execution-session state
            -> Moderari replay state
            -> Vestigare child trace_id
```

Before Replay model execution begins, Repetere must confirm that the Replay session
identity exists at the Pontis authority boundary and is propagated unchanged through
the effective execution path. Absence, replacement or loss of that identity is an
infrastructure failure and must stop the run before uncorrelated model execution.

### 11.3 Source Model Provenance and Replay Intent

Before requesting a model, Repetere must establish what model the Replay is intended
to use. The source Trace must provide sufficient authoritative provider/model
provenance to distinguish:

- the model actually used by the source execution;
- the provider actually used by the source execution;
- a researcher-requested alternate Replay model, where deliberate substitution is
  supported; and
- the currently selected runtime model, which must not silently replace either of
  the above.

If the source Trace lacks trustworthy model/provider provenance, Repetere must not
claim same-model reproduction. The staged run must expose that the execution
condition is unresolved and require an explicit supported model choice or fail as
`FAILED / INCOMPLETE`, according to the final M0.1 operator workflow.

Repetere must never infer the required model merely from Moderari configuration, a
preferred model, a currently resident Ollama model or the last model used by another
session.

### 11.4 Praebere Discovery, Selection and Readiness Prerequisite

Repetere must coordinate Replay model prerequisites through Nuntius with Praebere as
the authority. Provider-specific discovery or Ollama lifecycle logic must not be
implemented inside Repetere or Fiducia.

Before starting Vestigare recording or performing Replay model interaction,
Repetere must:

1. obtain the authoritative Praebere model/runtime state through Nuntius;
2. determine whether the required Replay model is available;
3. determine whether the current runtime-global selection is unlocked or already
   locked;
4. request the required model selection through Nuntius when selection is permitted;
5. require positive acknowledgement of the effective selected model;
6. require the N9.6 provider-neutral readiness result showing that the model is usable,
   not merely configured, selected or listed as locally available;
7. establish/activate the Replay execution session using its fresh Pontis
   `session_id`; and
8. begin Vestigare recording and Replay model execution only after every prerequisite
   has succeeded.

Success must never be inferred from silence, an HTTP timeout, an available-model list
or a selected-model name without readiness confirmation.

### 11.5 Required Model Unavailable

If the source or explicitly requested Replay model is not available through
Praebere:

- Repetere must not silently substitute the preferred, selected, resident or any
  other available model;
- model execution must not begin;
- an empty or misleading Replay Trace must not be presented as a completed
  reproduction;
- the run must be recorded as `FAILED / INCOMPLETE` with the required model,
  discovered provider state and authoritative failure reason;
- the staged Experiment and all previous run evidence must remain recoverable;
- a later retry must create a new Replay run and a new isolated Replay session;
- Fiducia must record the scheduled execution outcome accurately rather than treating
  request acceptance as Replay success.

Any automatic retry policy belongs to Fiducia scheduling configuration. A retry must
not conceal the original unavailable-model outcome or overwrite its evidence.

### 11.6 Runtime Locked to Another Model

If an ordinary or Replay execution session has locked Model A and a new Replay
requires Model B, Repetere must not attempt to override the runtime-global lock and
must not execute the Replay using Model A.

For M0.1, the required outcome is explicit non-execution:

``` text
runtime locked to required model
    -> Replay may proceed using that model

runtime locked to a different model
    -> Replay must not begin model execution
    -> scheduled/manual run remains deferred or becomes FAILED / INCOMPLETE
       according to the final orchestration policy
```

The final choice between a bounded deferred/queued state and immediate
`FAILED / INCOMPLETE` must be made explicitly in Fiducia/Repetere orchestration. It
must not become indefinite invisible waiting. The state must expose:

- required Replay model;
- currently authoritative selected model;
- lock state;
- active execution-session count where available;
- whether the run is waiting, declined or terminal; and
- any retry/deadline policy.

### 11.7 Concurrent Console and Replay Execution

An active Rogare, Pi or supported external-client session may coexist with an active
Replay. Concurrent execution is permitted only when every execution uses the same
authoritative runtime-global selected model.

The Replay session counts as an active execution session from its first model
interaction until it is explicitly deactivated/closed. While an ordinary session and
a Replay are active:

- model selection remains locked;
- both sessions use the same authoritative model;
- each session retains independent conversational and Moderari state;
- Replay reconstruction must not inherit live console context;
- console interaction must not inherit Replay context;
- responses, tool activity, control correlation and evidence must remain bound to
  the correct Pontis `session_id`; and
- completion of one session must not unlock model selection while the other remains
  active.

This concurrency is required particularly because Fiducia may start a scheduled
Replay while a researcher has an ordinary console session established or active.

### 11.8 Vestigare Single-Recording Arbitration

M0.1 permits only one active Vestigare recording across the installation. A Replay
run requires its own child Trace, so Repetere must obtain the recording boundary
before model execution.

- If a console session is active but no other Trace is recording, the Replay may
  acquire Vestigare recording and execute.
- Ordinary console traffic may continue while a Replay Trace is active only if live
  validation proves Vestigare records exclusively the Replay's bound Pontis
  `session_id`.
- If another console or Replay Trace is already active, a new Replay must not begin
  model execution without its required recording.
- The run must enter an explicit bounded waiting/deferred state or fail as
  `FAILED / INCOMPLETE`; it must not execute unrecorded and must not wait invisibly.
- Fiducia must preserve the scheduled trigger and resulting arbitration outcome.

### 11.9 Cleanup and Lock Release

Every terminal path must deactivate/close the Replay execution session and release
its contribution to the Praebere execution lock, including:

- matched completion;
- behavioural divergence;
- model/provider failure;
- Moderari prerequisite failure;
- Vestigare start failure;
- execution timeout;
- cancellation; and
- unexpected exception.

Cleanup must be idempotent. A crashed or failed Replay must not leave a phantom active
execution session that keeps model selection locked indefinitely. Conversely,
cleaning up the Replay session must not deactivate an unrelated console or Replay
session.

### 11.10 Fiducia Scheduled-Replay Outcomes

Fiducia must distinguish at least:

- scheduled and waiting for its execution time;
- waiting/deferred for the required model or recording boundary;
- accepted by Repetere;
- running;
- matched;
- diverged;
- `FAILED / INCOMPLETE` before model execution;
- `FAILED / INCOMPLETE` after partial infrastructure execution; and
- cancelled/expired according to an explicit scheduling policy.

Fiducia must not mark a scheduled run successful merely because Repetere accepted the
request asynchronously. The authoritative terminal Replay result and child Trace
relationship must be associated with the Fiducia execution record.

### 11.11 Required Cross-Service Acceptance

Before this work is complete, validate at least:

1. every Replay run receives a fresh Pontis-authoritative `session_id`;
2. the same identity reaches Praebere, Moderari, Repetere and the child Vestigare
   Trace;
3. same-source-model Replay with no competing active session;
4. required model available but idle/non-resident, using N9.6 readiness;
5. required model unavailable, with no substitution and no model execution;
6. runtime already locked to the Replay's required model;
7. runtime locked to a different model, with explicit defer/failure and no
   substitution;
8. active console execution and Replay execution concurrently using the same model;
9. independent responses, context, tool activity and session-correlated evidence for
   the concurrent console and Replay;
10. model selection remains locked until both execution sessions have ended;
11. active console Trace when a scheduled Replay becomes due;
12. active Replay Trace while unbound console traffic continues;
13. concurrent traffic does not contaminate the Replay child Trace;
14. every success, failure, timeout and cancellation path releases only the Replay's
    execution-session state; and
15. Fiducia records both asynchronous acceptance and the authoritative terminal
    outcome without overwriting prior failed-run evidence.

------------------------------------------------------------------------

## 12. Implementation Boundary

These changes should be implemented together with the planned
**child-of-staged Replay work** rather than as isolated UI patches.

That work should address as one coherent change:

1.  Replay Trace parent/child relationship;
2.  multiple Replay runs beneath a staged experiment;
3.  asynchronous state refresh;
4.  Trace metadata refresh;
5.  terminal result refresh;
6.  stale error-banner removal;
7.  behavioural-failure versus infrastructure-failure presentation;
8.  removal of ordinary lifecycle status pills;
9.  coloured-text status presentation; and
10. `Run again` lifecycle consistency.

------------------------------------------------------------------------

## 13. Exit Condition

The child-of-staged work is complete when an operator can start or rerun
a staged Replay and observe the complete lifecycle without manually
refreshing the browser:

``` text
Staged
   ↓
Running
   ↓
Matched · Completed
```

or:

``` text
Staged
   ↓
Running
   ↓
Failed · Completed
   ↓
Behavioural fork evidence
```

while the corresponding Replay-created Vestigare Trace is visibly
associated with that run and automatically progresses from recording to
its authoritative completed state.

No valid `running` state should generate an operation-failure banner,
and ordinary Replay/Trace status should be presented as coloured text
rather than status pills.

In addition, each Replay run must have one fresh Pontis-authoritative session identity,
must establish its required model through the Nuntius/Praebere authority and N9.6
readiness path before execution, must never silently substitute a model, and must
coexist safely with ordinary sessions using the same locked runtime-global model.
Unavailable models, conflicting model locks and Vestigare recording contention must
produce explicit recoverable orchestration states. Every terminal path must preserve
run evidence and release only that Replay's execution-session state.
