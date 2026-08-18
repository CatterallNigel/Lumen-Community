# Lumen Fiducia — Initial Development Architecture

**Status:** Development Planning  
**Date:** 10 August 2026  
**Position in Roadmap:** After Lumen Rogare, before Lumen Assess

---

## 1. Purpose

Lumen Fiducia is the orchestration layer responsible for building confidence in observed AI behaviour through repeated and scheduled evidence gathering.

Its initial responsibility is deliberately narrow:

> **Fiducia determines when a Replay should run and how many times it should run. Replay performs each individual run.**

This responsibility previously risked being incorporated into Lumen Replay itself through requirements such as **“run this replay X times.”**

That capability does not belong in Replay.

Replay should execute a single behavioural replay and report what occurred. Decisions about repetition, scheduling, evidence accumulation and eventually the sufficiency of that evidence belong to Fiducia.

This preserves the Lumen architectural principle that each component should have a clear and bounded responsibility.

---

## 2. Architectural Decision

### Replay executes once

A Replay invocation represents exactly **one replay experiment**.

Replay is responsible for:

- accepting a Replay Plan;
- executing that plan once;
- comparing expected and observed behaviour;
- identifying a Fork Point when divergence occurs;
- recording the resulting behavioural evidence;
- coordinating the associated Trace lifecycle;
- completing and returning the result of that individual replay.

Replay is **not** responsible for:

- executing itself repeatedly;
- maintaining a requested run count;
- scheduling future executions;
- deciding when another replay should occur;
- determining how much evidence is sufficient;
- interpreting multiple runs as a larger experiment;
- determining whether accumulated evidence establishes trust.

Any existing specification or roadmap requirement for Replay to support **“run X times”**, repeated execution, or scheduled execution should therefore be removed.

---

## 3. Fiducia's First Responsibility

The first Fiducia capability will be **Replay orchestration**.

Fiducia should be able to define an activity such as:

> Run Replay A five times.

or:

> Run Replay B according to this schedule.

Fiducia then invokes Replay separately for every required execution.

Conceptually:

```text
Fiducia Activity
      |
      +---- Replay Run 1
      |
      +---- Replay Run 2
      |
      +---- Replay Run 3
      |
      +---- ...
      |
      +---- Replay Run N
```

Each child execution remains an ordinary, independent Replay.

Fiducia owns the relationship between those runs.

---

## 4. Initial Fiducia Model

The first implementation should introduce the concept of a **Fiducia Activity**.

A Fiducia Activity describes an evidence-gathering operation and may initially contain:

```text
Activity
    id
    name
    replay_id
    requested_runs
    completed_runs
    schedule
    status
    created_at
    started_at
    completed_at
```

The exact persistence model will be determined during implementation.

The important architectural distinction is ownership:

```text
Replay owns:
    one execution

Fiducia owns:
    the orchestration of executions
```

---

## 5. Run Count

A basic Fiducia activity may request:

```text
Replay: authentication-regression
Runs:   10
```

Fiducia is responsible for invoking Replay ten times.

Replay itself sees ten independent requests.

It does not need to know:

- that ten runs were requested;
- which numbered run it is executing;
- whether previous runs matched;
- whether another run will follow.

This avoids coupling Replay to experimental policy.

---

## 6. Scheduling

Fiducia should also own scheduled Replay execution.

Examples may eventually include:

```text
Run every hour.

Run once each night.

Run after a new model deployment.

Run every Monday.

Run five times every 24 hours.
```

The initial implementation does not necessarily need to support every scheduling model.

The architectural requirement is simply:

> **Scheduling belongs to Fiducia, never Replay.**

Replay remains callable independently at any time for a single experiment.

---

## 7. Evidence Sets

Repeated Replay execution creates something that a single Replay cannot provide:

**an evidence set.**

For example:

```text
Fiducia Activity: Qwen Behaviour Stability

Replay Run 1    MATCH
Replay Run 2    MATCH
Replay Run 3    FORK
Replay Run 4    MATCH
Replay Run 5    MATCH
```

Fiducia owns the grouping of these observations.

This distinction will become important when Lumen Assess is introduced.

A Replay result answers:

> What happened during this execution?

A Fiducia evidence set answers:

> What behaviour have we repeatedly observed?

Assess can subsequently answer:

> What does that evidence mean?

---

## 8. Relationship with Assess

Fiducia should be developed **before Assess**.

The development sequence becomes:

```text
Rogare
   |
   v
Fiducia
   |
   v
Assess
```

This is intentional.

When Assess development begins, the system should already be capable of producing both:

1. individual Replay observations; and
2. collections of related observations produced by Fiducia.

Assess can therefore be designed around real evidence structures rather than assuming that every behavioural assessment concerns a single isolated Replay.

The responsibilities remain distinct:

```text
Replay
    observes behaviour

Fiducia
    gathers and organises evidence

Assess
    evaluates evidence
```

---

## 9. Longer-Term Fiducia Direction

Replay orchestration is Fiducia's **first responsibility**, not necessarily its final scope.

The broader purpose of Fiducia is suggested by its name:

**Fiducia — trust / confidence.**

The long-term objective is not simply to run experiments repeatedly.

It is to allow Lumen to develop calibrated confidence from accumulated behavioural evidence.

A useful principle is:

> **Replace constant verification with calibrated trust built from evidence.**

And more broadly:

> **Trust isn't the absence of verification. Trust is what accumulated evidence allows you to stop verifying every time.**

Repeated Replay execution provides the raw observations from which that principle can eventually become operational.

---

## 10. Future Direction: Adaptive Verification

A later Fiducia capability may move beyond fixed schedules and fixed run counts.

Instead of being instructed:

```text
Run this Replay 10 times.
```

Fiducia may eventually be able to determine that additional evidence is—or is not—required.

For example:

```text
Behaviour has remained stable across sufficient observations.
Reduce verification frequency.
```

or:

```text
Recent behaviour has diverged.
Increase verification frequency.
```

or:

```text
Confidence has fallen below an acceptable threshold.
Request additional Replay evidence.
```

This would turn Fiducia from a simple scheduler into an evidence-driven **verification policy engine**.

That capability should not be implemented prematurely, but the initial architecture should avoid preventing it.

---

## 11. Relationship with Trace

Fiducia should not duplicate Trace responsibilities.

The existing behavioural chain remains conceptually:

```text
Fiducia
    |
    v
Replay
    |
    v
Trace
    |
    v
Lumen
    |
    v
Model
```

Fiducia requests experiments.

Replay controls each experiment.

Trace records what happens.

Fiducia groups the resulting evidence.

Assess subsequently interprets it.

---

## 12. Servire Integration

Fiducia should eventually be exposed through Lumen Servire in the same manner as the other operational components.

Servire should provide the human-facing controls required to:

- create Fiducia activities;
- select a Replay;
- specify a run count;
- configure a schedule;
- start or stop an activity;
- inspect activity status;
- inspect individual Replay runs belonging to an activity;
- navigate from an activity into the associated Replay and Trace evidence.

Servire remains the operational control plane.

Fiducia remains responsible for orchestration and policy.

---

## 13. Initial Development Boundary

The first Fiducia milestone should remain intentionally small.

### Required

- Create a Fiducia activity.
- Associate the activity with an existing Replay.
- Specify the number of requested runs.
- Invoke Replay once for each requested run.
- Track individual executions.
- Track activity progress.
- Associate Replay results with the originating Fiducia activity.
- Expose sufficient status for later Servire integration.

### Architecturally Required

The design must allow future scheduled execution without placing scheduling logic into Replay.

### Not Initially Required

The first milestone does not need:

- behavioural assessment;
- trust scoring;
- semantic equivalence analysis;
- adaptive verification;
- confidence thresholds;
- automatic decisions about evidence sufficiency;
- continuity graph analysis;
- autonomous remediation.

Those capabilities should emerge only once the evidence model and Assess responsibilities are properly understood.

---

## 14. Component Responsibility Summary

```text
Lumen Replay
------------

"Run this experiment once."

Owns:
    execution
    comparison
    fork detection
    individual result


Lumen Fiducia
-------------

"Run this experiment according to this verification policy."

Initially owns:
    repetition
    scheduling
    orchestration
    evidence grouping


Lumen Assess
------------

"Determine what the observed evidence tells us."

Owns:
    evaluation
    sufficiency
    behavioural assessment
    interpretation
```

---

## 15. Architectural Invariant

The following should be treated as a Fiducia architectural invariant:

> **Lumen Replay executes a single replay experiment. Lumen Fiducia determines when Replay is executed, how often it is executed, and how related executions are grouped as evidence. Lumen Assess determines what that evidence means.**

This boundary should be preserved as the three components evolve.

---

## 16. Revised Development Order

Following the current Lumen component standardisation work, the planned sequence is:

```text
Lumen / Moderari standardisation
        |
        v
Servire alignment
        |
        v
Pontis integration
        |
        v
Rogare
        |
        v
Fiducia
        |
        v
Assess
```

Introducing Fiducia before Assess ensures that Assess is developed against a system already capable of producing meaningful collections of behavioural evidence rather than being designed solely around individual replay executions.

---

## 17. Guiding Principle

The architectural distinction can ultimately be reduced to three questions:

> **Replay:** What happened this time?

> **Fiducia:** What have we observed over time?

> **Assess:** What does that evidence tell us?

Keeping those questions separate preserves the single-responsibility architecture of the Lumen stack while providing a natural path from observation, through accumulated evidence, to calibrated trust.