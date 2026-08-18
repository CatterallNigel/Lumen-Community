# Lumen Praebere / Servire Integration

Servire 0.8.7 introduces Lumen Praebere as the managed implementation of the `model_provider` role.

## Boundary

Servire contains no Ollama-specific control logic. It starts and stops Praebere as a managed component and invokes the generic lifecycle endpoints configured for that component. Praebere owns all knowledge of Ollama, model load/unload, provider process ownership, and provider-specific status.

## Startup

1. Servire performs its normal non-mutating validation.
2. Praebere starts first.
3. Servire invokes Praebere `POST /lifecycle/start`.
4. Praebere ensures the provider is running and the configured model is loaded.
5. Servire health-gates Praebere and then starts Moderari and the remaining managed components in dependency order.

## Shutdown

Praebere is stopped last. Before terminating the Praebere component process, Servire invokes `POST /lifecycle/stop`. Praebere unloads the configured model and stops Ollama only when the Ollama process is owned by Praebere. An externally started Ollama process remains running.

## Configuration

Praebere runs on port 11431. Moderari retains port 11436 and has a 120-second health timeout to accommodate very cold startup conditions. Praebere lifecycle hooks use a 180-second ceiling.

Ollama is no longer represented as a Servire external dependency. It is an implementation detail behind the Praebere boundary.
