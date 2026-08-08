# LUMEN_SERVIRE_INIT_PLAN

## Purpose

Lumen Servire is the operational control layer for the Lumen stack. Its
purpose is to start, stop, monitor and coordinate the runtime components
from a single interface while remaining outside the client → model
traffic path.

Servire does **not** proxy model traffic, modify prompts, replay
conversations or assess behaviour. Those responsibilities remain with
the existing Lumen components.

## Architecture

### Runtime path

``` text
Client → Trace → Replay → Lumen → Model Provider
```

Servire is not inserted into this path.

### Managed stack (Lumen++)

-   Lumen
-   Replay
-   Trace

### External prerequisites

-   Client (currently Pi)
-   Model provider (currently Ollama)
-   MongoDB

These must be available before Lumen++ can start.

## Startup

Startup order:

1.  Verify Pi
2.  Verify Ollama
3.  Verify MongoDB
4.  Start Lumen
5.  Wait for Lumen health
6.  Start Replay
7.  Wait for Replay health
8.  Start Trace
9.  Wait for Trace health
10. Declare READY

Shutdown is always the reverse order.

## Rollback

Rollback is mandatory.

If startup fails after one or more services have started, Servire
performs an orderly reverse shutdown to return the stack to a known
state.

Example:

-   Lumen started
-   Replay started
-   Trace failed

Rollback:

-   Stop Replay
-   Stop Lumen

## Service definition

Each managed service should define:

-   Display name
-   Mode (managed/external)
-   Working directory
-   Python executable (virtual environment)
-   Launch arguments
-   Environment variables
-   Health endpoint
-   Dependencies
-   Startup timeout

Servire should invoke the virtual-environment Python executable directly
rather than activating a shell.

## Configuration viewer (Milestone 4)

Read-only viewer showing:

-   Working directory
-   Python executable
-   Launch command
-   Configuration file
-   Health endpoint
-   Dependencies

Preflight validation:

-   Directory exists
-   Executable exists
-   Config readable
-   Entry point exists
-   Port available

## Unified logging

Features:

-   Merged chronological log
-   Source-coloured badges
-   Severity highlighting
-   Component filters
-   stdout/stderr filters
-   Pause auto-scroll
-   Bounded in-memory history

Structured JSON logs should be parsed when possible with fallback to
plain text.

## Stack states

Overall:

-   STOPPED
-   BLOCKED
-   STARTING
-   READY
-   DEGRADED
-   FAILED
-   STOPPING

## Milestones

### M1 Foundation

FastAPI, dashboard, typed config, health endpoint.

### M2 Service Catalogue

Managed/external services, dependency graph, launch definitions.

### M3 Process Control

Start/stop/restart, PID tracking, stdout/stderr capture.

### M4 Configuration Inspection

Read-only config viewer and preflight validation.

### M5 Lifecycle

Dependency-aware startup, reverse shutdown, mandatory rollback.

### M6 Unified Logs

Merged logs, colour coding, filtering.

### M7 Operational Integration

Trace controls, Replay visibility, links to component UIs.

Assess remains outside the operational stack.

## Guiding Principles

-   Servire is the control plane.
-   Runtime ownership remains with Lumen, Replay and Trace.
-   Pi and Ollama are current implementations, not permanent
    dependencies.
-   Startup must be deterministic.
-   Rollback is mandatory.
-   Configuration should be visible before execution.
-   One merged operational log should present the entire stack.
