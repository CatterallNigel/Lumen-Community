# Lumen Pontis M6.4 Acceptance — Downstream Session Identity Preservation

## Purpose

Preserve the authoritative Rogare/Lumen session identity across the Pontis → downstream OpenAI-compatible request boundary so Lumen Moderari can maintain conversational continuity for sparse Rogare turns.

## Problem

Rogare supplies its stable session identity in the `X-Lumen-Session-Id` HTTP header. Pontis correctly used that identity for provider binding and capability enrichment, but the downstream JSON request body did not necessarily contain `session_id`. Moderari therefore could not resolve the stable client session and created a new conversation root for each sparse Rogare request.

## Change

When Pontis enriches a bound client request with the provider-owned model/tool snapshot, it now also writes the resolved authoritative identity into the downstream JSON payload as:

```json
{"session_id": "<resolved-session-id>"}
```

Header resolution remains authoritative. If a conflicting/stale `session_id` exists in the incoming body, the resolved header identity replaces it.

## Invariants

- Pontis remains the provider/session arbitration bridge.
- Rogare still owns no tools.
- ACP provider binding/session reuse is unchanged.
- Model/tool capability enrichment is unchanged apart from preserving session identity alongside it.
- Moderari remains responsible for conversational context reconstruction and continuity.

## Verification

- pytest: 53 passed
- Python compileall: passed
- ruff/mypy unavailable in the execution environment; run normal local gates before promotion.
