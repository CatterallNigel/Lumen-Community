# Lumen Praebere ↔ Moderari Future Integration Design

## Purpose

This document records the intended future architecture for integrating **Lumen Praebere** more deeply with **Lumen Moderari**, particularly around provider/model readiness, provider abstraction, and eventual model selection in Rogare.

The key principle remains:

> **Lumen components should know Lumen interfaces; provider adapters should know external providers.**

Praebere should become the source of truth for model-provider capability, while Moderari remains responsible for orchestrating model requests.

---

## Architectural Direction

The preferred responsibility boundary is:

```text
Rogare
   ↓
Pontis
   ↓
Moderari
   ↓
Praebere
   ↓
External Model Provider
   ├── Ollama
   ├── LM Studio
   └── Future providers
```

### Rogare

Rogare remains a conversational client.

It should not know whether:

- the model is currently loaded,
- the provider is Ollama, LM Studio, or another runtime,
- the provider needs to be started,
- the model needs to be loaded,
- or any provider-specific lifecycle details.

Later, Rogare may expose model selection to the user, but it should select from provider-neutral capability information exposed through Lumen rather than directly understanding external providers.

### Pontis

Pontis remains the client/session/tool-provider bridge.

It should not take responsibility for model-provider readiness or model lifecycle.

### Moderari

Moderari is the correct place for the **model readiness gate** because it owns model-request orchestration.

Before forwarding a model request upstream, Moderari should be able to ask Praebere:

- which provider/model is active for this request,
- whether the provider is available,
- whether the requested/configured model is available,
- whether the model is currently loaded,
- and, if required, ask Praebere to make the model ready.

Moderari should not contain Ollama-specific or LM-Studio-specific logic.

### Praebere

Praebere should become the source of truth for:

- provider identity,
- provider state,
- configured/selected model identity,
- model availability,
- model residency/loading state,
- provider/model lifecycle operations,
- and provider-specific implementation details.

Praebere should hide whether readiness requires:

- starting Ollama,
- loading a model,
- connecting to LM Studio,
- selecting a runtime endpoint,
- or another provider-specific action.

---

## Model Residency and Readiness

The overnight Lumen test showed an important operational behaviour:

A model may no longer be resident even though the provider remains available.

This distinction must be preserved:

```text
Provider running ≠ Model loaded
```

Praebere should therefore treat model residency as runtime state rather than assuming that a model loaded during Stack Start remains loaded indefinitely.

This allows external providers to reclaim memory when idle without breaking Lumen.

---

## Proposed Future Readiness Contract

A provider-neutral Praebere operation should be introduced, conceptually:

```text
POST /lifecycle/ensure-ready
```

or:

```text
POST /model/ensure-ready
```

The exact endpoint name can be decided when implemented.

Its semantics should be:

1. Determine the required provider/model.
2. Determine provider state.
3. Start the provider if required and permitted.
4. Determine model availability.
5. Load the model if required.
6. Verify the model is ready for inference.
7. Return provider/model readiness state to Moderari.

Conceptually:

```text
Incoming model request reaches Moderari
        ↓
Moderari asks Praebere for readiness
        ↓
Provider/model ready?
   yes ─────────────────→ Forward request
   no
        ↓
Praebere establishes readiness
        ↓
Verify ready
        ↓
Moderari forwards request
```

This should be idempotent.

If the provider and model are already ready, no lifecycle action should occur.

---

## Provider and Model Source of Truth

The current architecture still contains some duplicated provider/model configuration between components.

The future direction should be:

```text
Praebere = provider/model source of truth
Moderari = consumer of provider/model capability
```

Moderari should eventually obtain from Praebere:

- provider identifier,
- model identifier,
- model endpoint/capability information,
- readiness state,
- and any Lumen-level metadata required for request routing.

Moderari should not need to know how that provider implements lifecycle or residency.

---

## Suggested Future Praebere Status Contract

A future provider-neutral response may resemble:

```json
{
  "provider": {
    "id": "ollama",
    "state": "running",
    "managed": true
  },
  "model": {
    "id": "qwen2.5-coder:14b-32k",
    "state": "loaded",
    "ready": true
  }
}
```

The exact schema should be designed when the Moderari/Praebere interface is implemented.

Provider-specific fields should not leak into Moderari unless they are explicitly represented as generic Lumen capability metadata.

---

## Relationship to Rogare Model Selection

Rogare model selection should come **after** the Praebere ↔ Moderari provider/model contract is established.

The preferred sequence is:

### Phase 1 — Readiness Integration

Moderari asks Praebere to ensure the configured model is ready before inference.

### Phase 2 — Provider/Model Authority

Praebere becomes the authoritative source for the active provider/model.

Moderari removes duplicated provider/model lifecycle knowledge.

### Phase 3 — Provider/Model Discovery

Praebere exposes available providers/models through a provider-neutral Lumen interface.

Examples might eventually include:

```text
Ollama
  ├── qwen2.5-coder:14b-32k
  ├── devstral:24b
  └── gemma3:4b

LM Studio
  └── locally available models
```

### Phase 4 — Rogare Model Selection

Rogare can present available model choices to the user.

Rogare selects a Lumen model capability, not an Ollama or LM Studio implementation directly.

Conceptually:

```text
Rogare model choice
        ↓
Lumen model identifier
        ↓
Moderari
        ↓
Praebere resolves provider/model
        ↓
Provider adapter
```

---

## Important Architectural Rule

Model selection should describe **intent**, not provider implementation.

For example, Rogare may eventually request:

```text
model = qwen2.5-coder:14b-32k
```

but should not need to encode:

```text
provider = ollama
endpoint = 127.0.0.1:11434
load via /api/generate
```

Those remain Praebere/provider-adapter concerns.

---

## Operational Behaviour After Idle

If an external provider unloads a model during an idle period:

```text
Lumen remains running
Ollama remains running
Qwen becomes non-resident
```

the next user request should behave as:

```text
Rogare sends request
        ↓
Moderari receives request
        ↓
Moderari asks Praebere to ensure readiness
        ↓
Praebere detects model not loaded
        ↓
Praebere reloads model
        ↓
Moderari continues request
```

The expected user-visible consequence is a slower first response after a long idle period, rather than a failed request.

---

## Why Moderari Owns the Readiness Gate

Moderari is the best location because it is already responsible for model-request orchestration.

Putting readiness checks in Rogare would incorrectly make the client responsible for model infrastructure.

Putting them in Pontis would mix client/tool-provider bridging with model-provider lifecycle.

Keeping the readiness decision in Moderari and the readiness implementation in Praebere preserves separation of concerns:

> **Rogare expresses conversational intent.**  
> **Moderari orchestrates model requests.**  
> **Praebere resolves and maintains model-provider capability.**

---

## Future Work Summary

- Add a provider-neutral `ensure-ready` operation to Praebere.
- Allow Moderari to query Praebere before forwarding model requests.
- Make Praebere authoritative for provider/model identity and readiness.
- Remove duplicated provider/model configuration from Moderari where appropriate.
- Support automatic reload after provider-driven idle eviction.
- Add provider/model discovery to Praebere.
- Add additional provider adapters such as LM Studio.
- Only then expose provider-neutral model selection in Rogare.
- Preserve all provider-specific lifecycle knowledge inside Praebere adapters.

---

## Status

**Documented for future development.**

This is not required for the current Praebere M1/M1.1 or current Rogare work, but defines the intended architectural progression toward a fully provider-agnostic Lumen model layer.
