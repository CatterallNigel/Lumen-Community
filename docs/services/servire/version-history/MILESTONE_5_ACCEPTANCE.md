# Lumen Servire — Milestone 5 Acceptance

## Milestone

**M5 — Lumen++ Lifecycle**

Version: `0.5.0`

## Objective

Operate Lumen, Replay and Trace as one deterministic managed stack while enforcing external prerequisites, health-gated startup and mandatory rollback.

## Delivered

- Explicit stack lifecycle state machine:
  - `STOPPED`
  - `VALIDATING`
  - `READY_TO_START`
  - `STARTING`
  - `READY`
  - `DEGRADED`
  - `STOPPING`
  - `RESTARTING`
  - `FAILED`
- No boolean stack lifecycle flags.
- Runtime validation of required external roles.
- Initial external probes:
  - Pi — running-process probe.
  - Ollama — HTTP availability probe.
  - MongoDB — TCP reachability probe.
- Failed validation makes stack Start unavailable.
- Start revalidates immediately before launch.
- Dependency-derived managed startup order.
- Startup order for the current Lumen++ catalogue:
  1. Lumen
  2. Lumen Replay
  3. Lumen Trace
- Managed health polling after each process starts.
- Next service is not started until the previous service is healthy.
- Mandatory reverse-order rollback after any startup failure.
- Reverse-order stack shutdown.
- Stack restart implemented as orderly stop followed by validated startup.
- Stack-level operational log messages, including:
  - `Bringing Lumen++ online...`
  - dependency availability checks
  - health waits
  - READY state
  - rollback activity
- Stack Health panel now reports:
  - lifecycle state
  - managed service count
  - external availability count
  - last startup duration
  - last error
- External service cards display availability after validation.
- Stack actions are state-aware:
  - Validate
  - Start
  - Stop
  - Restart

## Startup Contract

A full stack start follows this contract:

```text
Validate configuration
        ↓
Verify Client
Verify Model Provider
Verify Persistence
        ↓
Bringing Lumen++ online...
        ↓
Start Lumen → wait healthy
        ↓
Start Replay → wait healthy
        ↓
Start Trace → wait healthy
        ↓
Lumen++ READY
```

No managed component starts if validation fails.

## Rollback Contract

Rollback is mandatory.

If any managed service fails to launch or fails its health gate, all services started during that attempt are stopped in reverse dependency order.

Example:

```text
Lumen READY
Replay READY
Trace FAILED
      ↓
Stop Trace
Stop Replay
Stop Lumen
      ↓
Lumen++ STOPPED
```

A partially started stack is never treated as READY.

## Scope Boundaries

M5 does not implement:

- full log filtering (M6)
- Trace/Replay operational integration (M7)
- client/model runtime selection
- model discovery from the provider
- editable configuration
- clickable configuration conveniences
- Assess integration

The following post-roadmap usability items remain explicitly required:

- Open Working Directory
- Open Configuration File
- Open Health Endpoint
- Copy Launch Command

## Quality Gates

Required locally:

```text
pytest
ruff check .
mypy src tests
```

Coverage threshold remains at least 95%.
