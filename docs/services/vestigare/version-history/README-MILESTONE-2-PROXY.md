# Milestone 2 — Transparent Proxy

This increment adds transparent HTTP forwarding from Lumen Trace to Lumen.

## Scope implemented

- Catch-all HTTP forwarding for GET, POST, PUT, PATCH, DELETE, OPTIONS and HEAD.
- Preserves path, raw query string, request body, end-to-end headers, response status and response body.
- Streams raw downstream response chunks rather than buffering model output.
- Removes only HTTP hop-by-hop headers; these are connection-specific and cannot be forwarded by an intermediary.
- Returns HTTP 502 when Lumen cannot be reached.
- Creates and closes one shared `httpx.AsyncClient` through the FastAPI lifespan.

## Not implemented

- Recording.
- MongoDB message writes.
- Recording controls.
- Live UI.
- WebSocket proxying.

## Runtime arrangement

- Pi -> Trace: `127.0.0.1:11435`
- Trace -> Lumen: `127.0.0.1:11436`
