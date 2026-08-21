# Servire Managed Component Clear Logs

Servire v0.8.6 adds operational **Clear Logs** actions for managed Lumen components.

## Component action

Each stopped managed component exposes Start, Stop, Restart, and Clear Logs. `Clear Logs` is disabled while the component is starting, running, or stopping.

Servire invokes the component's own offline maintenance command using its configured launch target, appending `clear-logs`. For module-based components this is equivalent to:

```text
python -m <component_module> clear-logs
```

The component remains responsible for its own log ownership and safety checks.

## Stack action

The Stack Actions panel also exposes **Clear Logs**.

It is enabled only when no managed Lumen component is running. It sequentially invokes the component-owned clear-log operation for every managed component in the configured stack.

This action clears managed component logs only; it does not clear Servire's own active log because Servire must remain running to service the UI action.

Servire's own logs can still be cleared offline with:

```text
python -m lumen_servire clear-logs
```
