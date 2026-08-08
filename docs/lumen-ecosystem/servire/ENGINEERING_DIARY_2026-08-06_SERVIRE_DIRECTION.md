# ENGINEERING_DIARY_2026-08-06_SERVIRE_DIRECTION

## Summary

With Lumen Replay reaching Milestone 10, development has been
intentionally frozen.

The Replay UI is considered functionally complete for its current
purpose. While there are future usability improvements that could be
made, they are now lower priority than completing the wider Lumen
operational ecosystem.

The next active project will be **Lumen Servire**.

------------------------------------------------------------------------

## Replay Status

Replay now provides the complete operational workflow:

``` text
Inspect → Stage → Start → Review
```

The UI now supports:

-   named replay experiments
-   duplicate name prevention
-   replay staging
-   replay execution
-   replay status presentation
-   matched and divergent replay presentation
-   replay lifecycle management

This represents a suitable point to freeze Replay and avoid unnecessary
refinement before Servire exists.

------------------------------------------------------------------------

## Architectural Decision

Servire is a **separate Lumen project**.

It does not become another proxy in the client → model traffic path.

Runtime traffic remains:

``` text
Client → Trace → Replay → Lumen → Model Provider
```

Servire exists outside this path as the operational control plane.

------------------------------------------------------------------------

## Initial Scope

Servire will initially manage the Lumen++ runtime:

-   Lumen
-   Replay
-   Trace

External dependencies are currently:

-   Pi
-   Ollama
-   MongoDB

These remain external services.

Servire verifies they are available before starting Lumen++ but does not
initially own their lifecycle.

This keeps Lumen client-agnostic and model-provider agnostic.

------------------------------------------------------------------------

## Startup Policy

Startup must be deterministic.

Current startup sequence:

1.  Verify Pi
2.  Verify Ollama
3.  Verify MongoDB
4.  Start Lumen
5.  Wait for health
6.  Start Replay
7.  Wait for health
8.  Start Trace
9.  Wait for health
10. Declare READY

Shutdown occurs in reverse dependency order.

------------------------------------------------------------------------

## Rollback

One important design decision made today:

Rollback is mandatory.

If startup fails after one or more managed services have started,
Servire performs an orderly reverse shutdown to return the stack to a
known state.

Partial startup is not considered a successful outcome.

------------------------------------------------------------------------

## Configuration Visibility

Servire will provide a read-only configuration viewer showing:

-   working directory
-   virtual-environment Python executable
-   launch command
-   configuration file
-   dependencies
-   health endpoint

This is intended as an operational aid before startup and during failure
investigation.

------------------------------------------------------------------------

## Unified Logging

Servire will merge stdout and stderr from managed services into one
chronological stream.

The interface will include:

-   source-coloured component badges
-   severity highlighting
-   filtering by component
-   filtering by severity
-   stdout/stderr filters
-   bounded in-memory history

This should replace multiple command windows with a single operational
view.

------------------------------------------------------------------------

## Milestone Plan

The project roadmap has been defined through seven milestones:

1.  Foundation
2.  Service Catalogue
3.  Process Control
4.  Configuration Inspection
5.  Lumen++ Lifecycle
6.  Unified Logging
7.  Operational Integration

Assess intentionally remains outside the operational stack.

------------------------------------------------------------------------

## Reflection

Replay has matured from an experimental proxy into a dedicated
behavioural replay engine.

Servire now addresses a different problem entirely: operational
simplicity.

Rather than extending Replay further, the next phase of development
focuses on reducing operational friction across the complete Lumen++
stack while maintaining clear architectural boundaries between
components.
