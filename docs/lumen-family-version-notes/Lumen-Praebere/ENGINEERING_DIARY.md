# Lumen Praebere Engineering Diary

## 2026-08-15 — M1 initial implementation

Created the first Lumen Praebere codebase as an independent model-provider component. M1 introduces a provider-neutral service/API, an Ollama adapter, separate process lifecycle controller, YAML configuration, operational logging, offline `clear-logs`, and tests. Ollama model operations use its HTTP API. Praebere remains outside the model traffic path and Servire is unchanged.


## 2026-08-15 — M1 provider ownership refinement

Real Ollama validation confirmed status detection and model unload, and exposed that a pre-existing Windows Ollama desktop instance must remain externally owned. Praebere now refuses provider stop when `managed_process=false`, removes external process-kill configuration, and only stops provider processes that Praebere itself started. The Praebere service port is standardized on 11431.

## 2026-08-15 — Model operation timeout hardening

Real managed-provider validation showed that a cold load of the configured Qwen model can exceed the normal provider HTTP timeout. Praebere now separates fast provider/status requests from long-running model operations. `request_timeout_seconds` remains 10 seconds, while `model_operation_timeout_seconds` defaults to 120 seconds and is used for Ollama load/unload requests. Ollama HTTP timeouts during load/unload are translated into controlled Praebere operation results (`success=false`, `changed=false`) rather than escaping as unhandled HTTP 500 responses.
