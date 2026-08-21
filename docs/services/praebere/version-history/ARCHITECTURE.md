# Lumen Praebere M1 Architecture

Praebere provides the Lumen-side abstraction for external model-provider control.

## Principle

**Lumen components should know Lumen interfaces; provider adapters should know external providers.**

M1 contains one adapter: Ollama. `PraebereService` and the HTTP API use only provider-neutral operations and state. Ollama HTTP endpoint details remain in `providers/ollama.py`.

## Lifecycle boundary

Praebere itself and Ollama have independent lifecycles. Starting/stopping the Praebere process must not implicitly start/stop Ollama. Provider lifecycle changes happen only through explicit `/operations/start` and `/operations/stop` requests.

## State

Provider: `stopped | running`.

Configured model: `unavailable | available | loaded`.

## Ollama M1 mapping

- availability/running probe: `GET /api/tags`
- installed model availability: `GET /api/tags`
- loaded state: `GET /api/ps`
- load: `POST /api/generate` with configured negative `keep_alive`
- unload: `POST /api/generate` with `keep_alive: 0`
- provider start: configured process command, initially `ollama serve`
- provider stop: terminate only the Praebere-owned process; externally managed providers are never terminated by Praebere

Praebere does not pull models in M1.


## Provider ownership invariant

Praebere may use an already-running external provider, but it does not acquire ownership of that provider process merely by detecting or using it. `managed_process=false` means provider start is idempotent and model load/unload remain available, while provider stop is refused. Only a process started by Praebere may be stopped by Praebere.


## Managed component lifecycle (M1.1)

Praebere exposes two aggregate lifecycle hooks for an operational control plane:

- `POST /lifecycle/start` ensures the provider is running and the configured model is loaded.
- `POST /lifecycle/stop` unloads the configured model and stops the provider only when Praebere owns the provider process.

These hooks preserve provider ownership. If Ollama was already running externally, lifecycle stop unloads the configured model but leaves the external provider process running. Servire therefore requests only the component lifecycle action; Praebere retains all provider-specific knowledge.
