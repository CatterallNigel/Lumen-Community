# LUMEN_SERVIRE_MILESTONES

## Milestone 1 -- Foundation

### Objective

Establish the Servire service and operator dashboard.

### Deliverables

-   FastAPI application
-   Typed configuration
-   Structured logging
-   Health endpoint
-   Basic operator dashboard
-   Servire self-status

### Acceptance Criteria

-   Servire starts cleanly
-   Dashboard accessible
-   Health endpoint reports healthy
-   Configuration loads successfully

------------------------------------------------------------------------

## Milestone 2 -- Service Catalogue

### Objective

Describe the managed stack and external dependencies.

### Deliverables

-   Managed vs external services
-   Dependency graph
-   Working directories
-   Python executables
-   Launch arguments
-   Health endpoints
-   Startup validation

### Acceptance Criteria

-   Services configurable
-   Invalid configuration detected before startup
-   Dependency graph displayed

------------------------------------------------------------------------

## Milestone 3 -- Process Control

### Objective

Control managed services individually.

### Deliverables

-   Start
-   Stop
-   Restart
-   PID tracking
-   stdout capture
-   stderr capture
-   Startup timeout
-   Failure reporting

### Acceptance Criteria

-   Each managed service can be controlled independently
-   Process state reflected in UI
-   Logs begin streaming immediately

------------------------------------------------------------------------

## Milestone 4 -- Configuration Inspection

### Objective

Provide read-only operational visibility.

### Deliverables

-   Configuration viewer
-   Working directory
-   Python executable
-   Launch command
-   Configuration file
-   Dependencies
-   Health endpoint
-   Preflight validation

### Acceptance Criteria

-   Operator can inspect configuration without editing
-   Missing files and invalid paths highlighted
-   Preflight clearly reports failures

------------------------------------------------------------------------

## Milestone 5 -- Lumen++ Lifecycle

### Objective

Operate the stack as a coordinated system.

### Deliverables

-   Verify prerequisites:
    -   Pi
    -   Ollama
    -   MongoDB
-   Ordered startup:
    1.  Lumen
    2.  Replay
    3.  Trace
-   Health-gated progression
-   Reverse shutdown
-   Mandatory rollback
-   Combined stack state

### Acceptance Criteria

-   Startup blocked if prerequisites unavailable
-   Startup follows dependency order
-   Rollback returns stack to STOPPED after failure
-   Stack state accurately reported

------------------------------------------------------------------------

## Milestone 6 -- Unified Logging

### Objective

Present one operational log.

### Deliverables

-   Chronological merged log
-   Source badges
-   Component colours
-   Severity highlighting
-   Filters:
    -   Component
    -   Severity
    -   stdout/stderr
-   Pause auto-scroll
-   Bounded in-memory history
-   JSON parsing with plain-text fallback

### Acceptance Criteria

-   All managed services appear in one stream
-   Filters work correctly
-   Log remains responsive during operation

------------------------------------------------------------------------

## Milestone 7 -- Operational Integration

### Objective

Coordinate existing Lumen components.

### Deliverables

-   Trace recording controls
-   Replay session visibility
-   Launch Replay operations
-   Links to Trace and Replay dashboards
-   Safe API-based operations only

### Acceptance Criteria

-   Recording can be started from Servire
-   Replay state visible
-   Native component ownership preserved

------------------------------------------------------------------------

# Out of Scope

The following are intentionally excluded:

-   Assess integration
-   Behaviour analysis
-   Prompt inspection
-   Model traffic proxying
-   Configuration editing
-   Docker/systemd orchestration
-   Pi ownership
-   Ollama ownership

------------------------------------------------------------------------

# Success Criteria

Servire should provide one operational interface for Lumen++,
eliminating multiple command windows while preserving loose coupling and
clear ownership between components.
