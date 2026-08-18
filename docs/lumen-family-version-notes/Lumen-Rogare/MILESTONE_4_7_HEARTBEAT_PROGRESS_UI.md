# Lumen Rogare M4.7 — Heartbeat / Progress UI Refinement

## Investigation result

The existing Moderari → Pontis → Rogare operational backchannel remains unchanged. Rogare continues to poll Pontis every five seconds while a conversational request is active. No evidence in the Rogare code indicates a transport failure.

The browser-side elapsed timer is reset by `setBusy(true)` at the start of each conversational send. It therefore measures the complete Rogare HTTP request lifetime, from submission until the final response/error/cancellation. It is not a session-age timer. A Moderari progress message can legitimately show a different elapsed value when that value describes a narrower downstream/model phase.

`lastHeartbeatAt` is Rogare receipt time: it is set when Rogare polls and consumes a retained heartbeat event. It is not claimed to be the exact Moderari emission timestamp.

Pontis `client_streams=0` is not interpreted by Rogare and is not part of the polling API contract. Rogare consumes retained session backchannel events with repeated HTTP GET requests rather than holding an event stream open, so that log field must not by itself be treated as evidence that the Rogare backchannel is disconnected. Confirmation of the precise Pontis field semantics belongs in Pontis itself.

## UI refinement

The activity panel now labels the local timer explicitly as `Request elapsed` and formats longer durations in minutes/hours. Heartbeat presentation is recency-based:

- before the first heartbeat: `Heartbeat: waiting`;
- after a heartbeat: `Heartbeat: active · last received <age> ago`.

The heartbeat age updates every second while the request is active. Moderari progress text remains the primary activity text and is not rewritten or moved into conversational history.

No backchannel transport, API, session identity, polling cadence, or event schema has been redesigned.
