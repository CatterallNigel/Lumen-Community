# Lumen Rogare M4.1 — Pontis Backchannel UI Integration

This focused increment wires the existing Rogare conversational UI to the Pontis operational backchannel without changing conversation, session, Replay, Trace, or other planned Rogare behaviour.

## Implemented

- Rogare server relays `GET /_pontis/sessions/{session_id}/backchannel` from Pontis through `GET /api/console/backchannel/{session_id}`.
- The browser primes a per-session event cursor when the Pontis provider session becomes ready.
- While a conversational request is active, Rogare polls Pontis once per second for events newer than the last observed event id.
- `progress` events update the existing **Lumen is working** activity panel with Moderari's progress message.
- `heartbeat` events visibly confirm that the backchannel is alive until the first richer progress event arrives.
- Operational events are transient UI state only; they are not inserted into Rogare conversation history.
- Polling stops when the model request completes or fails.

## Expected live path

`Moderari -> Pontis /_pontis/backchannel/events -> Rogare /api/console/backchannel/... -> existing activity UI`

Repetere and Vestigare are not involved in this path.
