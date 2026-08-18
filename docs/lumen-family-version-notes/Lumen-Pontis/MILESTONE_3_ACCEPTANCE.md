# Lumen Pontis — Milestone 3 Acceptance

## M3.5 objective

Validate the full dual-plane Servire-originated conversation lifecycle without imposing an artificial timeout on an active ACP prompt turn:

1. Pontis starts its transparent HTTP proxy on `server.host:server.port`.
2. Pontis starts `pi-acp`, creates a Pi ACP session, and confirms/selects the configured Pi model.
3. Pontis sends the natural-language bootstrap prompt `Calculate 2 + 2.` over ACP.
4. Pi begins its normal model-provider flow and sends OpenAI-compatible HTTP traffic back through Pontis.
5. Pontis forwards that HTTP traffic unchanged to the configured downstream target (currently Lumen can be used directly on port `11436`).
6. Pontis logs when each downstream streaming response starts and when it actually completes.
7. `session/prompt` is treated as a long-running agent turn and does not inherit the generic ACP request timeout when `acp.prompt_timeout_seconds` is `null`.
8. Pi may make one or more HTTP turns, including tool-call continuations, before the ACP `session/prompt` completes.
9. Pontis keeps the ACP adapter alive until the turn finishes, then tears down the test session and HTTP server cleanly.

M3.5 does not yet define explicit persistent ACP session-close semantics. That remains a later milestone.

## Required topology

For the current end-to-end test:

```text
Pontis --ACP--> Pi
                |
                | OpenAI-compatible HTTP
                v
             Pontis :11435
                |
                v
             Lumen :11436
                |
                v
              Qwen
```

Lumen must be running on the configured downstream address and able to reach Qwen before the test starts.

## Configuration

The relevant ACP timeout settings are:

```yaml
acp:
  request_timeout_seconds: 120.0
  prompt_timeout_seconds: null
```

`request_timeout_seconds` applies to ordinary ACP requests. `prompt_timeout_seconds: null` means an active `session/prompt` has no arbitrary wall-clock timeout. A positive number may be configured later if a deployment requires a hard limit.

## Quality gates

```powershell
pytest
ruff check .
mypy src tests
```

## Acceptance command

```powershell
python -m lumen_pontis acp-conversation --cwd C:\Development\Lumen-Pontis
```

The default prompt is:

```text
Calculate 2 + 2.
```

## Evidence to inspect

The expected lifecycle includes logs resembling:

```text
M3.5 sending ACP bootstrap prompt ...
HTTP proxy request method=POST path=/v1/chat/completions
M3.5 observed Pi entering HTTP plane ...
HTTP proxy response started ... status=200
HTTP proxy response completed ... status=200
...
ACP prompt completed ... stop_reason=...
```

If Qwen chooses to call a Pi tool, additional HTTP requests and ACP `session/update` events are expected before completion. M3.5 is successful when the complete turn can run without Pontis terminating `session/prompt` merely because it exceeds the generic 120-second ACP request timeout.
