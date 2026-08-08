# Replay Additional Logging

Replay now separates operational evidence from command-routing evidence.

## Operational log

The normal `logs/lumen-replay.log` records:

- operator command observed and whether Replay handled it;
- replay session and run start;
- each matching replay step;
- the first fork point with expected and observed summaries;
- transition into transparent passthrough;
- post-fork passthrough requests;
- terminal passthrough completion, cancellation, or full match.

## Command-decision log

`logs/lumen-replay-command-decisions.log` records every terminal `\\obt`
command seen at Replay's boundary. Each JSON line includes:

- method and path;
- raw terminal command;
- source role and message index;
- classification and parsed Replay action;
- replay ID when present;
- whether Replay handled it;
- whether it was forwarded to Lumen;
- the routing reason;
- Replay runtime state and active run ID.

Only the terminal conversational event is classified. Historical commands in a
request's accumulated context are not treated as fresh commands.
