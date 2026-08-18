# Milestone 10 Acceptance — Replay Operator Workflow

Milestone 10 completes the present Replay development cycle by making the operator workflow explicit and safe.

## Delivered

- Experiment names are mandatory for new prepared replays.
- Names are normalized and checked case-insensitively to prevent duplicate staged experiments.
- The dashboard presents a four-step workflow: inspect, stage, start, review.
- Prepared sessions can be started directly from the dashboard.
- Starting a run immediately marks its card `running` and disables Start/Run again and Unstage until the request resolves.
- Prepared sessions can be unstaged from the dashboard.
- The latest run status is associated with each staged experiment.
- Matched-step counts, explicit outcomes, failures, and first-fork expected/observed summaries are presented directly.
- Runtime configuration and historical capability cards are retained under a collapsible technical section.
- Legacy unnamed sessions remain readable but cannot be newly created.

## Architectural boundary

Replay remains an experiment controller and transparent proxy by default. Milestone 10 changes operator control and presentation only; it does not add assessment or alter the model traffic path.

## Version

`lumen-replay 0.10.3`

## Fork invariant

Forking ends comparison, never traffic. The fork-causing model response is returned unchanged to the live caller; Replay immediately becomes a transparent proxy, and Trace remains recording until the divergent branch reaches a terminal model response.


## UI Start Path Correction — 2026-08-10

- Start/Run again no longer calls `ReplayRuntime.start()` directly from the UI route.
- The UI emits the existing `\obt replay start <replay-id>` command through the
  configured normal stack ingress.
- Replay retains no Pontis-specific dependency; the ingress address is generic
  deployment configuration.
- The existing proxy command handler remains the sole runtime command execution
  path.
- Fork behaviour remains: persist divergence, end comparison, become transparent.

## M10.3 lifecycle/UI cleanup

- UI start result preserves a completed divergent outcome after the live post-fork branch reaches a terminal response.
- Automatic Trace stop treats HTTP 409/no-active-recording as an idempotent success because Trace may already have completed the recording while the terminal response traversed its proxy path.
- Trace recording deletion remains owned by Trace and is issued directly to the configured Trace control endpoint.
