# Milestone 2 — Proxy Observability Increment

This increment adds application-level logs for each proxied exchange without logging request or response payloads.

Events:

- `proxy_request_started`
- `proxy_upstream_response`
- `proxy_response_completed`
- `proxy_response_closed`
- `proxy_request_failed`

Every event includes an `exchange_id`. Completion is emitted only when the downstream response stream has finished; an interrupted or cancelled stream emits `proxy_response_closed` instead.

## Install

Extract into the root of the existing `Lumen-Trace` repository and overwrite the included files.

## Verify

```powershell
pytest -v
ruff check .
mypy
```

## Runtime example

```text
INFO lumen_trace.proxy: proxy_request_started exchange_id=... method=POST target=/v1/chat/completions
INFO lumen_trace.proxy: proxy_upstream_response exchange_id=... status=200 elapsed_ms=12
INFO lumen_trace.proxy: proxy_response_completed exchange_id=... method=POST target=/v1/chat/completions status=200 duration_ms=1834
```
