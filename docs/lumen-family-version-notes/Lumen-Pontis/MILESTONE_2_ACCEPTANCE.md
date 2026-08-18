# Milestone 2 - ACP Foundation

Milestone 2 establishes Pontis' second, separate communication plane while removing the temporary Pi HTTP payload inspection added during M1 validation.

## Acceptance criteria

- Pontis listens on the configured `server.host` / `server.port` (`127.0.0.1:11435` by default).
- HTTP traffic outside Pontis' own `/health` endpoint is handled by a transparent proxy and forwarded to the configured generic `downstream.host` / `downstream.port`.
- The HTTP proxy does not deserialize or interpret OpenAI conversational payloads.
- Request method, path, query string, body stream, and relevant headers are relayed downstream; downstream status, headers, and response stream are relayed back.
- ACP is implemented as a separate subsystem using newline-delimited JSON-RPC 2.0 over stdio.
- Pontis can launch the configured Pi ACP adapter (`pi-acp` by default).
- Pontis performs ACP `initialize` using protocol version 1.
- Pontis creates an ACP conversation using `session/new` and records the returned ACP `sessionId`.
- Pontis can issue `session/prompt` and collect `session/update` notifications, including `agent_message_chunk` text.
- Pontis has a session registry capable of binding a Servire/Lumen session ID to an ACP session ID without conflating this with Lumen's existing Pi-originated HTTP session resolution.
- `python -m lumen_pontis acp-smoke --cwd <absolute-or-relative-path>` exercises the ACP path using pi-acp's local `/session` command by default.
- Temporary full Pi HTTP request logging is removed.
- Tests, Ruff, and mypy are expected to be clean in the project Python 3.12.10 environment.

## Explicitly outside M2

- Servire integration.
- Trace/Replay integration beyond the generic downstream proxy target.
- A final design for delegating individual externally-originated model tool calls into Pi. M2 proves the ACP transport/session boundary first.
- Persistence of Pontis session bindings across Pontis restarts.
