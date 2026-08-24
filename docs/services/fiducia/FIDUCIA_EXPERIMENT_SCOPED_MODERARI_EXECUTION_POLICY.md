# Lumen Fiducia — Experiment-Scoped Moderari Execution Policy

**Status:** Future Development  
**Service:** Fiducia  
**Related Services:** Moderari, Repetere, Nuntius  
**Current Release Impact:** Not required for M0.1

## 1. Purpose

This document records a future refinement to the way Fiducia coordinates repeated Replay executions belonging to the same Experiment.

M0.1 deliberately requires each individual Replay to establish and positively confirm the required Moderari execution policy before the replay begins.

For system-prompt fidelity this means:

```text
Repetere
    |
    | \obt Moderari: Pass-through
    v
Nuntius
    |
    v
Moderari
    |
    | 200 OK — Pass-through applied
    v
Repetere
    |
    v
Replay begins
```

This is intentionally safe and explicit for M0.1.

When Fiducia coordinates multiple controlled repetitions of the same Experiment, however, repeatedly establishing the identical Moderari policy for every Replay is unnecessary control-plane repetition.

Future development should allow the required execution policy to be established once at Experiment scope.

## 2. Current M0.1 Behaviour

For M0.1, every Replay is independently responsible for establishing its prerequisites.

For an Experiment containing repeated executions:

```text
Fiducia
   |
   +-- Replay 1 -> Pass-through -> 200 -> execute
   |
   +-- Replay 2 -> Pass-through -> 200 -> execute
   |
   +-- Replay 3 -> Pass-through -> 200 -> execute
   |
   +-- Replay 4 -> Pass-through -> 200 -> execute
```

This produces additional control traffic, but it has an important property:

> **Every Replay independently proves that its required Moderari execution condition was established before model execution began.**

No Replay relies upon an assumption that configuration established for an earlier run is still active.

That behaviour should remain the M0.1 implementation.

## 3. Future Behaviour

Fiducia is the component that knows a collection of repeated executions belongs to one controlled Experiment.

It can therefore become responsible for establishing an Experiment-scoped execution policy before the first Replay begins.

Conceptually:

```text
Fiducia
    |
    | establish Moderari policy
    | scope = Experiment E-001
    v
Nuntius
    |
    v
Moderari
    |
    | 200 OK
    | policy established for E-001
    v
Fiducia
    |
    +-- Replay 1
    +-- Replay 2
    +-- Replay 3
    +-- Replay 4
    |
    | Experiment complete
    v
release / expire policy
```

The individual Replays no longer need to re-establish the identical policy provided the Experiment-scoped policy remains valid.

## 4. Architectural Principle

The optimisation must not weaken the reproducibility guarantee.

The future design must therefore **not** mean:

> Replay assumes Moderari is probably still in Pass-through.

Instead:

> **Fiducia explicitly establishes, positively confirms, owns and terminates an Experiment-scoped Moderari execution policy.**

The policy must have an explicit scope and lifecycle.

## 5. Policy Scope

The policy should be associated with an Experiment identity rather than becoming global Moderari state.

Conceptually:

```text
experiment_id: E-001
moderari_policy: pass-through
status: active
```

Only executions belonging to that Experiment may rely upon the policy.

Other concurrent sessions and Experiments must remain unaffected.

## 6. Positive Acknowledgement

The same acknowledgement rule established for M0.1 remains applicable.

Fiducia must not begin scheduling Replay executions that depend on the policy until Moderari has positively acknowledged that it has been applied.

Conceptually:

```text
200
    policy established -> Experiment runs may begin

204
    expected handler did not accept command -> do not begin

4xx
    policy rejected -> do not begin

5xx
    service failure -> do not begin

504 / timeout
    policy outcome unconfirmed -> do not begin
```

Success must never be inferred from silence.

## 7. Lifecycle

An Experiment-scoped policy requires an explicit lifecycle.

At minimum:

```text
REQUESTED
ACTIVE
RELEASED / EXPIRED
FAILED
```

The policy becomes usable only after `ACTIVE` has been positively established.

It must cease to apply when:

- the Experiment completes;
- the Experiment is cancelled;
- Fiducia explicitly releases it;
- its lease/lifetime expires;
- Moderari restarts or otherwise loses the scoped state.

The final mechanism should prevent abandoned Experiments from leaving persistent execution-policy state behind.

## 8. Lease / Expiry

A lease or equivalent bounded lifetime is preferable to an indefinite override.

Conceptually:

```text
Experiment E-001
Policy: Pass-through
Lease: active
```

Fiducia may renew the lease while the Experiment is active.

If Fiducia disappears or the Experiment terminates unexpectedly, the policy must eventually expire without requiring manual recovery.

The exact lease mechanism is future design work.

## 9. Replay Responsibility

With an active Experiment-scoped policy, Repetere remains responsible for reproducing an individual execution.

It should not become responsible for managing the Experiment-level policy lifecycle.

The responsibility split is:

```text
Fiducia
    coordinates Experiment
    establishes Experiment execution policy
    owns policy lifecycle

Repetere
    reproduces individual execution

Moderari
    applies policy for the defined Experiment scope

Nuntius
    routes commands and acknowledgements
```

## 10. Failure During an Experiment

Future design must define what happens if the Experiment-scoped policy becomes invalid while repeated executions are still outstanding.

The safe principle is:

> **No Replay should begin unless the required Experiment policy is known to remain valid.**

Possible future mechanisms include policy leases, policy-state checks or explicit invalidation notifications.

This should be designed when the feature is promoted into an active release rather than prematurely added to M0.1.

## 11. Trace Boundary

The Experiment-scoped control command remains Nuntius control-plane activity and should not become conversational Trace content.

Vestigare should continue to record the execution conditions that actually affected each model execution, including the effective system prompt.

The evidence must therefore remain sufficient to show what each Replay actually executed under, even though the control policy may have been established once for the Experiment.

## 12. Nuntius Diagnostics

The lifecycle of the Experiment-scoped policy should be visible in Nuntius diagnostics because it is control-plane behaviour.

Relevant events may include:

- policy request;
- Moderari acknowledgement;
- policy activation;
- renewal;
- release;
- expiry;
- failure;
- timeout;
- late response.

These events do not belong in Vestigare conversational Trace.

## 13. Why This Is Deferred

This refinement is not required to make M0.1 correct.

The M0.1 per-Replay acknowledgement model is deliberately repetitive but deterministic and easy to reason about.

Introducing Experiment-scoped policy now would add:

- policy lifecycle;
- lease/expiry behaviour;
- additional Experiment state;
- recovery semantics;
- new concurrency considerations.

Those are worthwhile once repeated Experiment execution is mature, but they are not necessary to establish the M0.1 research baseline.

## 14. Future Acceptance Direction

When this feature is promoted into a release, acceptance should demonstrate that:

- Fiducia can establish a Moderari policy for a specific Experiment;
- Moderari positively acknowledges the policy before any dependent Replay begins;
- multiple Replays can use the established policy without reissuing the same configuration command;
- unrelated sessions and Experiments are unaffected;
- policy loss prevents new dependent Replays from starting;
- completion/cancellation releases the policy;
- abandoned policy state expires safely;
- every Replay still records the effective execution conditions required for reproduction;
- policy lifecycle is visible through Nuntius diagnostics.

## 15. Summary

M0.1 should retain:

> **Replay -> establish policy -> acknowledge -> execute.**

Future Fiducia development may evolve this to:

> **Experiment -> establish and acknowledge scoped policy once -> execute controlled Replays -> release/expire policy.**

The purpose is to reduce unnecessary repeated control traffic without replacing explicit reproducibility with an assumption about persistent state.
