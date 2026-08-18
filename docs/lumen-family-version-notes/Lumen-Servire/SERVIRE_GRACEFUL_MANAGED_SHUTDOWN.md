# Servire Graceful Managed Shutdown — v0.8.9

Servire owns the lifecycle of services configured as `managed`. A normal ASGI shutdown, including Ctrl+C while running Servire from a terminal, now invokes the same dependency-aware Lumen stack stop sequence as the dashboard **Stop** action before Servire exits.

The stop order remains the reverse of managed startup order:

1. Rogare
2. Pontis
3. Vestigare
4. Repetere
5. Moderari
6. Praebere

Praebere retains ownership of its configured model-provider lifecycle. Therefore stopping Praebere through Servire invokes Praebere's managed `/lifecycle/stop` endpoint before the Praebere process is terminated, allowing it to stop its Ollama/model provider cleanly.

After the stack-aware stop attempt, Servire performs the existing `ProcessController.shutdown()` cleanup pass. Shutdown failures are logged but do not prevent the Servire application itself from terminating. External/operator-managed dependencies are not stopped by this path.
