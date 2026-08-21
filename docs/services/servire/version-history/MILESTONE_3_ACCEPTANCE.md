# Milestone 3 Acceptance — Process Control

## Objective

Give Servire direct lifecycle control over individually managed services without yet implementing full-stack orchestration.

## Delivered

- Individual Start, Stop and Restart controls for Lumen, Replay and Trace.
- Direct execution of each configured virtual-environment Python executable.
- Configured working-directory support.
- Script and module launch support.
- PID and runtime-state tracking.
- Process creation timeout and launch-failure reporting.
- Graceful termination with kill fallback after shutdown timeout.
- stdout and stderr capture.
- Source-labelled bounded in-memory operational log.
- Automatic child-process cleanup when Servire shuts down.
- External services remain operator-managed and cannot be launched by Servire.

## Explicitly deferred

- External dependency health probing.
- Dependency-aware Lumen++ startup.
- Stack-level Start, Stop and Restart.
- Health-gated progression and rollback.
- Full log filtering and live streaming controls.

These remain scheduled for later milestones.

## Acceptance Criteria

- Each managed service can be started independently.
- Each managed service can be stopped independently.
- A running managed service can be restarted.
- PID and state are visible on the operator dashboard.
- stdout/stderr are captured and labelled by source.
- Immediate process launch failures are visible to the operator.
- External services cannot be process-managed.
- Servire shutdown does not intentionally leave managed child processes running.
- pytest, Ruff and mypy quality gates pass with at least 95% coverage.
