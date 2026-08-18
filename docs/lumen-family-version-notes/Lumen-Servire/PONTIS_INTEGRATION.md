# Pontis Integration

Pontis is now a first-class managed Lumen++ component in Servire.

## Runtime integration

- Pontis working directory: `C:\Development\Lumen-Pontis`
- Launch command: `.venv\Scripts\python.exe -m lumen_pontis`
- HTTP/management port: `11435`
- Health endpoint: `http://127.0.0.1:11435/health`
- Pontis starts after Lumen, Replay, and Trace.
- Normal stack shutdown is the reverse order, so Pontis stops first.
- Pontis stdout and stderr are captured by the existing managed process log pipeline and appear in the Servire Operational Log with source `Lumen Pontis`.

## Workspace integration

Pontis has a Servire workspace tab even though Pontis does not currently expose a UI. While Pontis is healthy, the workspace displays `Pontis is running`. If Pontis is unavailable, the normal unavailable workspace is shown.

The interface definition uses `embed: false`. When Pontis gains a web UI, set `embed: true` and point its interface URL at that UI; Servire will then display it in the standard component iframe.

## Pi lifecycle transition

Pontis owns the headless Pi ACP runtime. Pi remains represented in Servire as the implementation of the `client` role, but it is no longer a required pre-start external dependency. This prevents Servire from requiring Pi to already be running before Pontis has had a chance to start it.
