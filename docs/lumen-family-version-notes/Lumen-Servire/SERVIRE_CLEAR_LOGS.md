# Lumen Servire — Clear Logs

Servire provides the same component-owned offline log maintenance command used across the Lumen components:

```text
python -m lumen_servire clear-logs
```

The command operates only on the Servire project-root `logs` directory. It preserves the directory itself, removes files, symlinks and nested directories beneath it, and refuses to run while Servire is listening on its configured host/port.

The maintenance command does not start the FastAPI application, stack controller or managed Lumen components.

Normal startup remains:

```text
python -m lumen_servire
```
