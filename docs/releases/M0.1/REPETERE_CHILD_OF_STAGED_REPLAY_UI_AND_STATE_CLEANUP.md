# Repetere --- Child-of-Staged Replay UI and State Cleanup

**Status:** Planned\
**Target:** M0.1 --- Repetere child-of-staged work\
**Recorded:** 2026-08-28\
**Scope:** Repetere / Vestigare integration and Repetere operator UI

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

## 11. Implementation Boundary

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

## 12. Exit Condition

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
