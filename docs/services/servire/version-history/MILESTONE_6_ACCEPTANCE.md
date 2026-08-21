# Lumen Servire — Milestone 6 Acceptance

## Milestone

**M6 — Unified Operational Logging & Live Operator Feedback**

Version: `0.6.0`

## Objective

Make Servire's Operational Log a practical live operations console and provide immediate UI feedback for long-running lifecycle actions.

## Delivered

- Operational events are returned newest-first.
- Server-side log history remains bounded by the ProcessController log capacity.
- New `/api/logs` endpoint exposes recent normalised events for live dashboard updates.
- Dashboard polls new operational events while open.
- Source colours remain consistent:
  - Lumen++ / Servire
  - Lumen
  - Replay
  - Trace
- Filters added for:
  - source
  - severity
  - stream (`system`, `stdout`, `stderr`)
  - message text search
- Pause freezes presentation while the server-side buffer continues receiving events.
- Resume immediately renders the latest buffered state.
- Clear removes the currently visible history without deleting the underlying ProcessController buffer.
- Structured JSON logs are normalised when possible.
- Plain-text logs remain lossless when structured parsing is unavailable.
- Severity is derived independently of stdout/stderr stream selection.
- stderr is not automatically treated as an error.
- Stack and individual service actions provide immediate pending labels:
  - `Validating...`
  - `Starting...`
  - `Stopping...`
  - `Restarting...`
- Action requests run asynchronously from the dashboard so live log polling can continue during startup/shutdown.
- Existing M5 state-machine, health gating and mandatory rollback semantics are unchanged.

## Log Ordering Contract

The newest event is always presented at the top of the operational log.

```text
10:31:00  Lumen++      SYSTEM  INFO  Lumen++ READY
10:30:59  Lumen Trace  SYSTEM  INFO  Lumen Trace READY
10:30:58  Lumen Trace  SYSTEM  INFO  Process started...
...
```

Operators no longer need to scroll to the bottom to see current activity.

## Pause / Clear Contract

`Pause` affects display only. Servire continues buffering new operational events.

`Clear` affects display only. It does not mutate or erase the underlying bounded operational history.

## Scope Boundaries

M6 does not implement:

- Trace/Replay operational integration (M7)
- editable configuration
- client/model runtime selection
- provider model discovery UI
- Assess integration
- persistent Servire log storage

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
