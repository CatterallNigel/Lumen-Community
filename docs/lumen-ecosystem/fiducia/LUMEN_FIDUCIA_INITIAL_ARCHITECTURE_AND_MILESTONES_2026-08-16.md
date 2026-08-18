# Lumen Fiducia --- Initial Architecture and Development Milestones

**Date:** 2026-08-16\
**Status:** Development architecture baseline\
**Development root:** `C:\Development\Lumen-Fiducia`\
**Python:** 3.12.10

This document defines the initial architecture and development sequence
for Lumen Fiducia. It reflects the current Lumen development roadmap and
establishes the responsibility boundary between Repetere, Fiducia, and
Aestimare.

> **Repetere executes one Replay Run.**
>
> **Fiducia decides when, how often and how many Replay Runs should
> occur.**
>
> **Aestimare determines what the resulting behavioural evidence
> means.**

The implementation must preserve this boundary.

## Repository Structure

Fiducia uses the standard project-root structure:

``` text
C:\Development\Lumen-Fiducia\
├── doc\
├── logs\
├── run\                  # runtime PID files where required
├── src\
├── tests\
├── config.yaml
├── pyproject.toml
├── README.md
└── ANY_VERSION_NOTES.md
```

The development environment uses Python 3.12.10 with a project `.venv`,
Git, pytest, pytest-cov, Ruff, and mypy.

------------------------------------------------------------------------

## 1. Position in the Stack

Fiducia does **not** enter the normal conversational request path.

Conceptually:

``` text
                         ┌──────────── Fiducia ────────────┐
                         │ scheduling / orchestration     │
                         │ evidence grouping              │
                         └──────────────┬─────────────────┘
                                        │
                                        │ supported API
                                        v
Servire → Rogare → Pontis → Vestigare → Repetere → Moderari → Praebere → Ollama/Qwen
                                        │
                                        └── Replay Run / Result
```

Fiducia communicates with **Repetere through Repetere's supported
service API**, not through Repetere's database, files, or internal
Python objects.

There should be no direct Fiducia relationship with Vestigare, Moderari,
Pontis, Praebere, Ollama, or Pi during the initial development phase.

Repetere already owns execution of the individual replay experiment and
coordination of the associated behavioural recording lifecycle. Fiducia
requests and organises those experiments.

------------------------------------------------------------------------

## 2. Core Domain Model

The initial Fiducia persistence model should use three explicit
entities:

``` text
FiduciaSchedule
    │
    ├── FiduciaExecution #1
    │       ├── ReplayRunReference #1
    │       ├── ReplayRunReference #2
    │       └── ReplayRunReference #3
    │
    ├── FiduciaExecution #2
    │       ├── ReplayRunReference #4
    │       ├── ReplayRunReference #5
    │       └── ReplayRunReference #6
    │
    └── ...
```

This distinction is required because schedule frequency and Replay Runs
per scheduled occurrence are independent concepts.

For example:

``` text
Run every:       24 hours
Runs each time:  10
```

means one Fiducia scheduled occurrence every 24 hours, with that
occurrence invoking Repetere ten separate times.

### 2.1 FiduciaSchedule

`FiduciaSchedule` represents the enduring orchestration policy.

Suggested initial fields:

``` text
id
name

prepared_replay_id
prepared_replay_name          # cached/display metadata only

schedule_type                 # once | recurring
first_run_at
recurrence_type               # interval initially
recurrence_interval
recurrence_unit               # minutes | hours | days

runs_per_execution

enabled
status                        # active | disabled | cancelled

next_run_at
last_run_at

created_at
updated_at
cancelled_at
```

The Prepared Replay identifier supplied by Repetere is authoritative.
Cached descriptive information belongs only to presentation/history and
must not become a duplicate source of truth.

### 2.2 FiduciaExecution

`FiduciaExecution` represents one occurrence of a Fiducia schedule.

For a schedule configured as:

``` text
Every 24 hours
Runs each time: 10
```

each 24-hour occurrence creates one `FiduciaExecution`, containing ten
individual Replay Run invocations.

Suggested initial fields:

``` text
id
schedule_id

scheduled_for
started_at
completed_at

runs_requested
runs_started
runs_completed
runs_failed

status
    scheduled
    running
    completed
    completed_with_failures
    failed
    cancelled

error
created_at
updated_at
```

A `FiduciaExecution` also provides a natural evidence-set boundary for
future use by Aestimare without introducing behavioural interpretation
into Fiducia.

### 2.3 ReplayRunReference

Fiducia retains references to Repetere-owned Replay Runs and Results. It
does not duplicate the Replay Result.

Suggested initial fields:

``` text
id
execution_id

ordinal                       # 1..N
repetere_run_id
repetere_result_id            # if separately exposed

requested_at
started_at
completed_at

status
    requested
    running
    completed
    failed

error
```

The ownership relationship is therefore:

``` text
Schedule
  → occurrence
      → Replay Run 1
      → Replay Run 2
      → Replay Run 3
```

Repetere remains unaware of the schedule or number of sibling Replay
Runs.

------------------------------------------------------------------------

## 3. Scheduling Model for M1

The initial implementation should deliberately avoid introducing cron
expressions.

The first scheduling model should support:

``` text
One time
    Run at: <datetime>

Recurring
    First run: <datetime>
    Every: <integer> <minutes|hours|days>

Runs each time:
    <integer>
```

This satisfies the initial scheduling requirement while keeping the
behaviour explicit, understandable, and straightforward to test.

Later scheduling capabilities may include:

``` text
weekly schedules
calendar schedules
cron-like schedules
deployment-triggered schedules
assessment-informed schedules
```

These can be introduced without changing the fundamental
schedule/execution model.

Scheduling belongs to Fiducia and must never migrate into Repetere.

------------------------------------------------------------------------

## 4. Scheduler and Execution Architecture

Scheduling and execution orchestration should be separate internal
responsibilities even though both belong to Fiducia.

``` text
                 REST / UI
                    │
                    v
             ScheduleService
                    │
                    v
              Persistence
                    ▲
                    │
             SchedulerEngine
                    │
        due schedule detected
                    v
            ExecutionService
                    │
             create execution
                    │
                    v
             ReplayClient
                    │
                    v
                 Repetere
```

A suitable source structure is:

``` text
src\
└── lumen_fiducia\
    ├── __init__.py
    ├── main.py
    │
    ├── api\
    │   ├── health.py
    │   ├── schedules.py
    │   ├── executions.py
    │   ├── replays.py
    │   └── logs.py
    │
    ├── config\
    │   └── settings.py
    │
    ├── domain\
    │   ├── schedule.py
    │   ├── execution.py
    │   ├── replay_run.py
    │   └── enums.py
    │
    ├── persistence\
    │   ├── database.py
    │   ├── repositories.py
    │   └── migrations.py
    │
    ├── services\
    │   ├── schedule_service.py
    │   ├── execution_service.py
    │   └── scheduler.py
    │
    ├── clients\
    │   └── repetere.py
    │
    ├── lifecycle\
    │   ├── runtime.py
    │   └── dependency_validation.py
    │
    ├── logging\
    │   ├── configuration.py
    │   └── management.py
    │
    └── ui\
        ├── routes.py
        ├── static\
        └── templates\
```

Exact module names can evolve during implementation. The architectural
requirement is separation between API, domain, persistence,
scheduling/execution services, lifecycle/log management, UI, and
communication with Repetere.

------------------------------------------------------------------------

## 5. Persistence Choice

Fiducia should use **MongoDB**, consistent with the existing Lumen persistence architecture.

Fiducia owns its own database namespace and must never depend upon another component's
internal collections.

Initial configuration:

```yaml
mongodb:
  uri: mongodb://127.0.0.1:27017
  database: lumen_fiducia
```

The initial domain collections are expected to be:

```text
fiducia_schedules
fiducia_executions
fiducia_replay_runs
```

The exact document structures should remain owned by Fiducia's persistence layer rather
than leaking into its API contract.

### Restart Recovery

Persistence across restart must include operational recovery rather than simply storing records.

On startup Fiducia should:

1. load enabled schedules;
2. determine their calculated next execution;
3. identify executions left in `running` state;
4. reconcile incomplete executions as far as the Repetere contract permits;
5. resume scheduling.

For the initial implementation, Fiducia should **not automatically backfill every occurrence
missed while the service was offline**.

Initial policy:

```text
service restarted after next_run_at:
    identify/log that an occurrence was missed
    calculate the next future occurrence
    do not silently backfill the missed occurrence
```

Catch-up behaviour can later become an explicit configurable policy.

---

## 6. Repetere Contract Required by Fiducia

Before substantial Fiducia implementation begins, the current Repetere
API should be inspected to determine whether the required supported
operations already exist.

Conceptually Fiducia requires:

``` text
GET prepared replays
GET prepared replay

POST execute prepared replay
GET replay run/status
GET replay result/status
```

The fundamental Repetere invocation must continue to mean:

``` text
prepared_replay_id = X
```

> Execute this Prepared Replay **once**.

Fiducia must not ask Repetere to understand:

``` text
runs = 10
frequency = 24h
fiducia_schedule = ...
```

An optional opaque correlation context may later be useful:

``` text
requested_by: fiducia
correlation_id: <fiducia-run-reference-id>
```

If introduced, Repetere must not interpret that metadata or use it to
alter execution. Its purpose would be provenance and cross-component
correlation only.

------------------------------------------------------------------------

## 7. Execution Semantics

Replay Runs within a single Fiducia occurrence should initially execute
**sequentially**.

Example:

``` text
Execution requires 5 runs

Run 1 → finishes
Run 2 → finishes
Run 3 → finishes
Run 4 → finishes
Run 5 → finishes
Execution completes
```

The initial implementation should not launch the five runs concurrently.

Sequential execution:

-   avoids competition for a local model/GPU;
-   simplifies state transitions and failure handling;
-   makes orchestration behaviour deterministic;
-   preserves Repetere's one-run-per-invocation responsibility.

A failed Replay Run should normally fail that individual run rather than
automatically abort all remaining runs in the occurrence.

Example:

``` text
Requested: 5

1 COMPLETED
2 COMPLETED
3 FAILED
4 COMPLETED
5 COMPLETED

Execution:
    COMPLETED_WITH_FAILURES
```

This preserves the complete evidence-gathering attempt.

------------------------------------------------------------------------

## 8. Proposed API

The initial REST surface should approximately provide:

``` text
GET    /health
GET    /status

GET    /api/replays
GET    /api/replays/{id}

GET    /api/schedules
POST   /api/schedules
GET    /api/schedules/{id}
PATCH  /api/schedules/{id}

POST   /api/schedules/{id}/enable
POST   /api/schedules/{id}/disable
POST   /api/schedules/{id}/cancel

GET    /api/executions
GET    /api/executions/{id}
GET    /api/executions/{id}/runs

POST   /api/logs/clear
```

A future convenience operation may be:

``` text
POST /api/schedules/{id}/run-now
```

This is not required for the first acceptance criteria and should be
deferred unless it proves useful during development/testing.

------------------------------------------------------------------------

## 9. Schedule States Versus Execution States

Schedule state, execution state, and individual Replay Run state must
remain distinct.

### Schedule States

``` text
ACTIVE
DISABLED
CANCELLED
```

`CANCELLED` is terminal.

A disabled schedule remains editable and can subsequently be enabled.

### Execution States

``` text
SCHEDULED
RUNNING
COMPLETED
COMPLETED_WITH_FAILURES
FAILED
CANCELLED
```

### Replay Run Reference States

``` text
REQUESTED
RUNNING
COMPLETED
FAILED
```

Keeping these state machines separate prevents a generic `status` field
from acquiring different meanings depending on context.

------------------------------------------------------------------------

## 10. Dependency Behaviour

At startup Fiducia should validate:

``` text
configuration
persistence
Repetere connectivity / compatibility
```

Failure handling should distinguish between dependencies.

### Persistence Failure

Fiducia cannot safely operate.

**Result:** startup failure.

### Invalid Configuration

Fiducia cannot safely operate.

**Result:** startup failure.

### Repetere Unavailable

Fiducia should start and report a **DEGRADED** state.

Schedules must remain inspectable and manageable while Repetere is
temporarily unavailable.

A scheduled occurrence that fires while Repetere is unavailable should
become a controlled failed execution rather than crashing Fiducia.

This behaviour also provides a clean basis for later Servire lifecycle
and dependency integration.

------------------------------------------------------------------------

## 11. Logging and Clear Logs

Fiducia owns its operational logs and must expose component-owned Clear
Logs functionality from the outset.

The standard root structure provides:

``` text
logs\
```

Fiducia is responsible for creating, writing, rotating/managing as
required, and clearing its own log files.

Servire must later invoke Fiducia's supported Clear Logs interface.

**Servire must never delete, truncate, or otherwise manipulate Fiducia
log files directly.**

Useful initial Fiducia operational events include:

``` text
FIDUCIA_STARTING
FIDUCIA_READY
REPETERE_AVAILABLE
REPETERE_UNAVAILABLE

SCHEDULE_CREATED
SCHEDULE_UPDATED
SCHEDULE_ENABLED
SCHEDULE_DISABLED
SCHEDULE_CANCELLED

EXECUTION_SCHEDULED
EXECUTION_STARTED
REPLAY_RUN_REQUESTED
REPLAY_RUN_COMPLETED
REPLAY_RUN_FAILED
EXECUTION_COMPLETED
EXECUTION_FAILED

LOGS_CLEARED
FIDUCIA_STOPPING
```

These events describe operational behaviour only. Fiducia must not log
behavioural interpretation as though it were an assessment.

------------------------------------------------------------------------

## 12. Standalone UI

The first stable Fiducia capability requires a usable standalone UI
before Servire integration.

It should provide three principal areas:

1.  **Available Prepared Replays**
2.  **Schedule Configuration**
3.  **Scheduled Work / Execution Status**

A simple initial layout is sufficient:

``` text
┌──────────────────────────────────────────────────────┐
│ Lumen Fiducia                         HEALTHY        │
├──────────────────────────────────────────────────────┤
│ AVAILABLE PREPARED REPLAYS                           │
│                                                      │
│ ○ Replay A       prepared ...                        │
│ ○ Replay B       prepared ...                        │
├──────────────────────────────────────────────────────┤
│ SCHEDULE CONFIGURATION                               │
│                                                      │
│ Name:             __________________                 │
│ Replay:           Replay A                           │
│ Type:             Recurring                          │
│ First execution:  __________________                 │
│ Run every:        [24] [hours]                       │
│ Runs each time:   [10]                               │
│ Enabled:          [✓]                                │
│                                      [Create]        │
├──────────────────────────────────────────────────────┤
│ SCHEDULED WORK                                       │
│                                                      │
│ Replay A Stability                                   │
│ Every 24 hours · 10 runs                             │
│ Next: ...                                            │
│ Last: 10 completed / 0 failed                        │
│ ACTIVE                                               │
└──────────────────────────────────────────────────────┘
```

The UI distinction between:

``` text
Run every:
```

and:

``` text
Runs each time:
```

is a UX invariant.

These controls represent different Fiducia concepts and must never be
combined or ambiguously labelled.

------------------------------------------------------------------------

## 13. Explicit M1 Exclusions

The initial Fiducia capability explicitly does **not** perform:

-   Replay match/fork determination;
-   behavioural scoring;
-   semantic comparison;
-   sufficiency judgement;
-   confidence percentages;
-   trust scoring;
-   adaptive scheduling;
-   model selection;
-   direct Vestigare evidence inspection;
-   direct model interaction;
-   automatic remediation;
-   Aestimare assessment logic.

Fiducia gathers and organises behavioural evidence.

It does not determine what that evidence means.

------------------------------------------------------------------------

## 14. Initial Development Milestones

The first development phase should be divided into small, independently
testable milestones.

### M0 --- Fiducia Foundation

Create the Fiducia service foundation using the established repository
and development environment.

Deliver:

``` text
FastAPI service
config.yaml
health/status
operational logging
component-owned Clear Logs
lifecycle startup/shutdown
run/PID handling where required
Repetere dependency configuration
MongoDB persistence foundation
pytest
pytest-cov
Ruff
mypy
README.md
ANY_VERSION_NOTES.md
```

**Acceptance:** Fiducia starts independently, initialises its
persistence, exposes health/status, owns its logs, and can clear those
logs through its own supported interface.

### M1 --- Repetere Discovery Contract

Implement Fiducia → Repetere communication.

Deliver:

``` text
discover Prepared Replays
retrieve/display Prepared Replay metadata
controlled Repetere-unavailable behaviour
```

This milestone determines whether any small supported API addition is
required in Repetere.

Any Repetere change must remain strictly an interface/exposure change.
Scheduling or repetition logic must not move into Repetere.

**Acceptance:** Fiducia can reliably identify the Prepared Replays
available for scheduling.

### M2 --- Schedule Domain and Persistence

Implement:

``` text
create schedule
read schedules
update schedule
enable
disable
cancel

one-time schedule definition
recurring interval definition
runs_per_execution

next_run_at calculation
restart persistence
```

Automatic execution is not required yet.

**Acceptance:** schedules can be created and managed, Fiducia can be
restarted, and the same scheduling state is recovered.

### M3 --- One-Time Execution

Introduce the scheduler/execution path.

Implement:

``` text
detect due one-time schedule
create FiduciaExecution
invoke Repetere once per requested run
persist Replay Run references
track completion/failure
```

This is the first complete end-to-end Fiducia orchestration behaviour.

**Acceptance:** schedule one Prepared Replay for five runs and observe
exactly five independent Repetere Replay Runs associated with one
Fiducia execution.

### M4 --- Recurring Scheduling

Add recurring execution.

Implement:

``` text
interval scheduling
next occurrence calculation
execution history
restart recovery
missed-occurrence policy
enable/disable behaviour
```

**Acceptance:** a recurring schedule creates separate Fiducia executions
over time, each containing the configured number of independent Replay
Runs.

### M5 --- Standalone UI

Implement the three initial UI areas:

1.  Available Prepared Replays
2.  Schedule Configuration
3.  Scheduled Work / Execution Status

**Acceptance:** the complete normal Fiducia scheduling workflow can be
performed without curl or separate API tooling.

### M6 --- Fiducia First Stable Capability

Perform the first hardening/stabilisation milestone.

Include:

``` text
API consistency
controlled error handling
dependency degradation
restart tests
scheduler race-condition tests
persistence tests
Clear Logs tests
UI workflow tests
pytest / coverage
Ruff
mypy
documentation
```

The first stable Fiducia capability is complete when:

-   Prepared Replays can be discovered through Repetere;
-   a user can create a schedule through the standalone UI;
-   the schedule defines when and how often it executes;
-   the schedule independently defines how many Replay Runs occur at
    each execution;
-   Fiducia invokes Repetere once for each required Replay Run;
-   execution state and resulting Replay references are retained;
-   schedules survive restart;
-   the service contract is stable;
-   tests and quality checks pass;
-   the standalone UI is usable.

At this point Fiducia development pauses for immediate Servire
integration rather than continuing into more sophisticated assurance
capabilities.

### M7 --- Servire Integration

Integrate Fiducia as a first-class Servire-managed component.

Implement:

``` text
Fiducia service configuration
dependency topology
startup/shutdown ordering
health/status
Clear Logs through the Fiducia API
Fiducia workspace/tab
embedded Fiducia UI
full-stack regression
```

Servire operates and presents Fiducia.

Servire does not absorb Fiducia's scheduling or orchestration
responsibilities.

------------------------------------------------------------------------

## 15. Development Sequence

The immediate implementation sequence is:

``` text
M0  Foundation
 ↓
M1  Repetere discovery
 ↓
M2  Schedule model + persistence
 ↓
M3  One-time execution
 ↓
M4  Recurring execution
 ↓
M5  Standalone UI
 ↓
M6  Stabilisation
 ↓
──────── First useful Fiducia boundary ────────
 ↓
M7  Servire integration
 ↓
Begin Aestimare architecture
```

This sequence gives each development increment a concrete, testable
outcome while implementing the current Fiducia roadmap without
prematurely introducing Aestimare responsibilities.

Before M1 implementation, the current Repetere API should be inspected
so that Fiducia consumes the existing supported Prepared Replay and
execution contracts wherever possible.

The first coding task is therefore **M0 --- Fiducia Foundation**.

------------------------------------------------------------------------

## Architectural Invariant

The following invariant applies throughout Fiducia development:

> **Repetere executes one Replay Run. Fiducia determines when, how often
> and how many Replay Runs should occur. Aestimare determines what the
> resulting behavioural evidence means.**

Convenience must not be allowed to migrate responsibility between those
components.
