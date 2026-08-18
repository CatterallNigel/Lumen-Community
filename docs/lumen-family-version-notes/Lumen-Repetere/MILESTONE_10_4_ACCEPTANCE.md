# Lumen Repetere M10.4 — Interrupted Run Recovery

## Purpose

Prevent a staged Replay from remaining permanently `running` after the Repetere process is stopped or restarted before the run reaches a terminal state.

## Behaviour

On application startup Repetere queries its own Replay-run store for every run still persisted with `status=running`. Process-local runtime state cannot survive a service restart, so those records are orphaned for the new service instance. Each orphaned run is transitioned to:

- `status=cancelled`
- `completed_at=<startup recovery time>`
- `failure_reason=Interrupted by Replay service restart`

The staged Replay session itself is preserved. The existing dashboard logic therefore presents the recovered terminal state and re-enables **Run again** and **Unstage**.

## Architectural boundary

Recovery only reconciles Repetere-owned persisted run state. It does not delete the staged Replay, manipulate another component's persistence, or infer a model outcome after an interrupted process.

## Acceptance

- Service startup recovers every persisted `running` run.
- Terminal runs are left unchanged.
- Recovered runs remain as evidence rather than being deleted.
- Recovered staged sessions can be run again or unstaged through the existing UI.
- Existing normal run lifecycle behaviour remains unchanged.

## Dependency-injection integrity

Startup interrupted-run recovery uses the same `ReplayRunRepositoryFactory` dependency-injection boundary as normal Replay runtime persistence. Application tests inject `FakeReplayRunRepository`; production continues to default to `MongoReplayRunRepository`. This prevents startup recovery from bypassing test/application repository wiring while preserving strict recovery semantics.

A lifespan-level regression test verifies that a persisted `running` run supplied through the injected repository is converted to `cancelled`, receives a completion timestamp and interruption reason, and retains its matched-step evidence.
