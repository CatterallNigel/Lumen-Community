# Replay Tool-Call Behaviour

Replay uses a Lumen Trace recording as an executable behavioural script.

## Comparison boundary

Replay compares only model tool calls visible to Pi:

- tool name;
- canonical JSON arguments;
- ordering of multiple calls in the same response.

Generated call identifiers are ignored. Assistant prose, progress output, reasoning text,
heartbeats, Lumen checkpoints, formatting, timing and final-answer wording are logged as
diagnostic evidence but cannot cause a fork.

## Matching path

When the live tool call matches the next recorded tool call, Replay advances to the next
recorded Pi-to-Lumen request. That request contains Pi's recorded tool result, so the result
is returned through Lumen to the model and replay continues.

## Fork path

When the live tool call differs in name, arguments or order from the next recorded tool
call, Replay records the first fork and becomes a transparent proxy. Trace then captures
the live path through Pi until the model produces its final response.

## Non-tool responses

Recorded and live non-tool responses are not comparison points. A live non-tool response
while Replay is waiting for a recorded tool call ends the private replay without a fork,
because Replay must not inject a recorded tool result for a tool the model did not call.
