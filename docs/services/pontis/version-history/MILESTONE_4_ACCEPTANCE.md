# Lumen Pontis — Milestone 4 Acceptance

## Objective

Move Pontis from a validated dual-plane prototype to an operational component that Servire can manage.

M4 keeps one long-lived `pi-acp` process alive for the Pontis runtime, keeps the HTTP proxy active at the same time, and manages multiple independent Servire/Lumen sessions over that shared ACP runtime.

## Operational lifecycle

```text
Pontis starts
    |
    +--> HTTP transparent proxy starts
    |
    +--> pi-acp / Pi ACP runtime starts once
    |
    `--> Pontis READY

READY
    |
    +--> create session A
    +--> create session B
    +--> prompt A / prompt B independently
    +--> logically close either session
    `--> continue remaining sessions

Pontis shutdown
    |
    +--> clear session bindings
    +--> stop ACP adapter / Pi
    `--> stop HTTP proxy
```

## Session states

Pontis exposes explicit bridge lifecycle state:

- `creating`
- `idle`
- `prompting`
- `closing`
- `closed`
- `failed`

Successful prompts return a session to `idle` and increment its prompt count. Prompt failures mark that session `failed`. If the long-lived ACP adapter disappears, Pontis is no longer ready and existing bindings are marked failed when accessed.

## Servire-facing management boundary

The operational HTTP service reserves `/_pontis/*` for Pontis management and leaves all other traffic on the transparent proxy path.

Available M4 endpoints:

```text
GET    /_pontis/status
GET    /_pontis/sessions
POST   /_pontis/sessions
GET    /_pontis/sessions/{session_id}
POST   /_pontis/sessions/{session_id}/prompt
DELETE /_pontis/sessions/{session_id}
```

The create-session request is:

```json
{
  "session_id": "servire-session-1",
  "cwd": "C:/Development/project"
}
```

The prompt request is:

```json
{
  "prompt": "Calculate 2 + 2."
}
```

These endpoints are the initial stable integration boundary for Servire. Pi-originated OpenAI-compatible traffic remains opaque and is not parsed or normalized by Pontis.

## Session close semantics in M4

`DELETE /_pontis/sessions/{session_id}` is a **Pontis logical close**: the Servire/Lumen-to-ACP binding is removed and cannot receive further prompts through Pontis.

M4 does not invent a non-standard ACP protocol command to destroy the corresponding Pi session. The underlying Pi session is reclaimed when the long-lived `pi-acp` process exits. Explicit ACP/Pi session disposal semantics can be refined later when required.

A session cannot be logically closed while its prompt is actively running.

## Normal operational command

```powershell
python -m lumen_pontis
```

Pontis starts both communication planes and remains running until shutdown.

## Multi-session acceptance command

With Lumen running at the configured downstream address:

```powershell
python -m lumen_pontis m4-acceptance --cwd C:\Development\Lumen-Pontis
```

The acceptance test:

1. starts Pontis once;
2. creates two separate ACP sessions on the same long-lived adapter;
3. prompts both sessions;
4. closes session A;
5. proves session B still exists;
6. sends a continuation on session B;
7. shuts Pontis down cleanly.

Expected final line:

```text
M4 acceptance: PASS
```

## Quality gates

```powershell
pytest
ruff check .
mypy src tests
```
