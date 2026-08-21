# Lumen Repetere — M10.5 Housekeeping and Runtime Finalisation

M10.5 tightens operator behaviour without changing Repetere's architectural responsibility.

## Accepted behaviour

- `python -m lumen_replay clear-logs` clears only `$root/logs` and refuses to run while the Replay service port is active.
- The Trace-recordings catalogue is contained in a fixed-height scrolling panel with a sticky header.
- Starting or re-running a staged Replay immediately resets the displayed **Matched steps** value to `0` before the request leaves the browser.
- A divergent terminal response no longer attempts to stop Trace while that same response is still travelling through Trace. Trace finalisation is deferred until Replay has delivered the HTTP response, preventing the stop/request circular wait seen in operational testing.
- A Trace recording already completed by the time deferred finalisation executes is treated as an idempotent success.

## Architectural boundary

Replay still owns only replay execution and the automatic Trace lifecycle associated with that execution. Trace remains the owner of recording persistence and completion state.

## Operator start-result classification correction

- The operator UI verifies that a staged Replay exists in Repetere before dispatching its start command through normal stack ingress.
- `replay-not-found` is reserved for that local staged-Replay lookup failure. A downstream HTTP `404` received after a valid Replay has been found and dispatched is not interpreted as "Replay not found".
- A new runtime outcome created by the current start request takes precedence over the downstream HTTP status. In particular, a divergent run remains reported as divergent even when a provider/control continuation subsequently returns an HTTP error.
- A prior run from an earlier invocation cannot be reused to classify the current start request; runtime outcomes are accepted only when their run id was created after the current UI start dispatch began.

## Final UI classification and quality-gate follow-up

- The operator UI now reserves `replay-not-found` for a genuine local staged-Replay lookup miss before dispatch.
- A downstream continuation HTTP error after a Replay has started cannot be reclassified as `replay-not-found`.
- New operator UI regression tests cover repository lookup failure and a newly-started running outcome so the new classification branches remain exercised without weakening the 95% coverage gate.
- Test imports are kept Ruff-sorted; no local `ruff --fix` step is required for the delivered package.

## Live divergent-run completion visibility

- A divergent Replay is presented as two independent dimensions: behavioural outcome (`divergent`) and lifecycle (`running` or `completed`).
- While a forked live continuation remains active, the Replay card shows `divergent · running`, Trace remains recording, and Run again / Unstage remain disabled.
- Repetere exposes a lightweight `/ui/replays/{replay_id}/status` operator endpoint. The dashboard polls active cards and updates them without requiring the operator to inspect Vestigare or infer completion from button state.
- When the terminal continuation settles and Trace completes, the card changes to `divergent · completed`, the operator notice explicitly reports completion, and Run again / Unstage are enabled.
- The page-level divergence notice is correlated to the Replay id that was started, so an older staged experiment cannot overwrite the current run's completion message.

## Staged Trace recording integrity

- A Trace recording referenced by a staged Replay is visibly marked `staged for replay` in the Trace recordings catalogue, including the staged experiment name when available.
- Stage is rendered disabled for an already-staged recording; the same Trace recording cannot be staged into a second Replay while the first staged reference exists.
- Delete is rendered disabled while a Trace recording is referenced by a staged Replay.
- These restrictions are enforced server-side as well as in the UI: preparation rejects an already-staged recording and the Trace delete route refuses to call Vestigare while a staged Replay references the recording.
- Unstaging the Replay removes the reference and makes the completed Trace recording eligible for staging or deletion again on the next dashboard refresh.

## Operator state consistency follow-up

- A staged Trace recording now presents a single operational status: **staged for replay**. The underlying completed state is not shown as a second competing badge while the recording is staged.
- A duplicate Start action that reaches Replay while the same experiment already has an active run is reported as **replay-running**, not **replay-not-ready**.
- Replay now enforces an internal outcome invariant: once a run has persisted its first behavioural divergence, that same run cannot later be reclassified as matched.
- The live-status endpoint remains the authoritative UI source for the run lifecycle and persisted run outcome.
