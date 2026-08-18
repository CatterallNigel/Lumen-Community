# Lumen Praebere Architecture

## Overview

Praebere is the model-provider lifecycle service for the Lumen stack.

It has a single responsibility:

**Manage the availability and lifecycle of the configured external model provider without owning conversational orchestration.**

Praebere currently manages the local Ollama provider and configured Qwen model required by the Lumen runtime.

---

# Architectural Position

```text
                         Servire
                            │
                    lifecycle control
                            │
                            ▼
                    Lumen Praebere
                            │
                   provider lifecycle
                            │
                            ▼
                         Ollama
                            │
                            ▼
                    Configured Model

Rogare → Pontis → Vestigare → Repetere → Moderari ──────┘
                         model request path
```

Praebere is not part of the conversational request path itself. It ensures that the provider required by that path is available and correctly managed.

---

# Provider Responsibility

Praebere owns provider lifecycle rather than model reasoning.

Its responsibilities include:

- starting the configured provider process when required;
- checking provider availability;
- discovering/confirming configured model availability;
- loading or warming the configured model where required;
- reporting provider/model operational state;
- stopping provider resources that Praebere itself manages;
- exposing lifecycle operations for Servire.

Praebere does not own:

- prompts;
- responses;
- conversation history;
- model reasoning policy;
- tool calls;
- session context;
- replay or assessment behaviour.

---

# Relationship to Moderari

Moderari is the model orchestrator.

Praebere is the model-provider lifecycle manager.

The distinction is:

```text
Moderari decides how the model is used.
Praebere ensures the configured model provider is available to be used.
```

Moderari sends conversational/model traffic to the configured provider endpoint.

Praebere does not inspect or mediate those conversational requests.

This keeps provider lifecycle independent from reasoning orchestration.

---

# Relationship to Servire

Servire is the operational control surface for the Lumen stack.

Servire starts and stops Praebere as a managed service and uses Praebere's lifecycle interface to coordinate the external provider.

The intended startup relationship is:

```text
Servire
   │
   ▼
Praebere starts
   │
   ▼
Provider lifecycle start
   │
   ├── ensure Ollama is running
   ├── verify provider API
   ├── verify configured model
   └── prepare/warm model as required
```

The intended shutdown relationship is:

```text
Servire shutdown / Stop
        │
        ▼
reverse managed stack shutdown
        │
        ▼
Praebere lifecycle stop
        │
        ├── release/unload managed model resources
        └── stop Praebere-managed Ollama provider
        │
        ▼
Praebere process stops
```

Servire should not have to understand Ollama-specific lifecycle mechanics. Those belong to Praebere.

---

# Managed Versus External Provider Ownership

Praebere must distinguish resources that it manages from resources that merely happen to be available.

If Praebere starts the provider process, it may stop that provider as part of its lifecycle shutdown.

An externally managed provider should not be terminated merely because Praebere or Servire stops unless configuration explicitly establishes that ownership.

This ownership boundary prevents Lumen lifecycle operations from unexpectedly terminating unrelated provider processes.

---

# Ollama Provider Lifecycle

For the current local provider configuration, Praebere starts Ollama using the provider command equivalent to:

```text
ollama serve
```

Praebere then uses the Ollama HTTP API to establish availability and model state.

Operational checks include provider/model endpoints such as:

```text
/api/tags
/api/ps
```

Model preparation may invoke the configured model through the provider so that it is available for subsequent Lumen requests.

The configured provider and model are operational configuration, not conversational state.

---

# Cold Start Behaviour

A cold provider/model start can be materially slower than normal model inference.

During cold startup the provider may allocate substantial memory while loading the configured model. Once initialization has completed, observed memory use can settle to a lower steady-state level.

This means a Rogare session bootstrap can remain in `Establishing provider session` while the provider/model is being prepared even though the Lumen services themselves are healthy.

Praebere owns the provider lifecycle portion of this process; Pontis and Rogare own their respective session/bootstrap presentation responsibilities.

---

# Health and Availability

Praebere separates process existence from provider readiness.

A running Praebere process does not by itself prove that the configured model provider is usable.

Operational state should therefore distinguish, where applicable:

- Praebere service running;
- provider process running;
- provider API reachable;
- configured model available;
- configured model loaded/ready.

This allows Servire to present meaningful operational state rather than equating process existence with model availability.

---

# Shutdown Semantics

Praebere is the correct component to perform provider shutdown because it knows:

- which provider is configured;
- how that provider was started;
- whether Praebere owns the provider process;
- which model is configured;
- how provider/model resources should be released.

Servire therefore requests lifecycle shutdown from Praebere rather than directly killing Ollama.

Provider shutdown can take longer than ordinary Lumen service termination. Servire allows a dedicated lifecycle timeout for Praebere before falling back to its final process cleanup behaviour.

If provider shutdown repeatedly approaches or exceeds that lifecycle allowance, the issue belongs in Praebere/provider lifecycle investigation rather than being hidden by continually increasing Servire's timeout.

---

# Configuration Boundary

Praebere receives provider/model configuration rather than discovering conversational requirements itself.

The current architecture supports a configured provider such as Ollama and a configured model such as Qwen.

Future model selection or provider discovery can extend this contract, but responsibility should remain separated:

- Praebere reports provider/model capability and lifecycle state;
- Moderari determines model orchestration requirements;
- Rogare may present available choices to the user;
- Servire manages operational lifecycle.

Praebere should not become a conversational policy engine.

---

# Runtime Responsibilities

Praebere owns:

- provider process lifecycle;
- configured model-provider availability;
- provider health checks;
- model availability/state checks;
- provider startup and shutdown;
- provider ownership tracking;
- lifecycle API;
- operational logging and health.

Praebere does not own:

- conversational sessions;
- prompts or responses;
- context management;
- model behavioural orchestration;
- ACP communication;
- tool execution;
- trace;
- replay;
- assessment.

---

# Architectural Invariants

Praebere manages providers; it does not orchestrate conversations.

Moderari must remain provider-agnostic at the lifecycle level and should not acquire Ollama process-management responsibilities.

Servire coordinates lifecycle but should delegate provider-specific start/stop behaviour to Praebere.

Praebere must only terminate provider resources that fall within its configured management ownership.

Provider/model lifecycle state must remain distinct from Lumen conversational/session state.

This separation allows the underlying model supplier to change in the future without moving provider-specific lifecycle behaviour into Moderari, Rogare, Pontis, or Servire.
