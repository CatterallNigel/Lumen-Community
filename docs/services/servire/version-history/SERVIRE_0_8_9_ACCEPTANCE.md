# Lumen Servire 0.8.9 — Graceful Managed Stack Shutdown Acceptance

## Change

Ctrl+C / ASGI application shutdown now requests an orderly stop of the managed Lumen stack before Servire exits.

## Required behaviour

- Application shutdown uses the stack lifecycle controller when managed components are running.
- Managed components stop in reverse dependency/startup order.
- Praebere's managed lifecycle stop hook executes before its process is terminated, allowing the configured model provider to be stopped/unloaded.
- External dependencies are not stopped.
- The existing process-controller shutdown pass remains as final cleanup.
- An exception or incomplete component stop is logged and does not prevent Servire itself from completing shutdown.
