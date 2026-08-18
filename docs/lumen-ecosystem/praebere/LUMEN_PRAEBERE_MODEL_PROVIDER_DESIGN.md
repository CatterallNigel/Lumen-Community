# Lumen Praebere --- Model Provider Control

## Status

**Planned Lumen component**

Praebere is to be implemented after the current Lumen Servire update is
complete.

## Name and Responsibility

**Lumen Praebere --- Model Provider**

*Praebere* is used in the sense of **to provide, supply, furnish, or
make available**.

Praebere will provide the Lumen-side abstraction between Lumen and
external model providers. Its role is comparable, at the model-provider
boundary, to the separation Pontis provides at the client/tool-provider
boundary.

The architectural principle is:

> **Lumen components should know Lumen interfaces; provider adapters
> should know external providers.**

Servire should therefore not contain Ollama-specific lifecycle logic.
That responsibility belongs to Praebere.

------------------------------------------------------------------------

## Initial Scope

The first implementation of Praebere is intentionally narrow.

It will support:

-   **Ollama** as the only model provider;
-   the **configured Qwen model** as the only model Praebere is required
    to manage;
-   provider status;
-   provider start;
-   model load;
-   model unload;
-   provider stop.

There is no requirement in the first milestone for general model
discovery, model selection, multiple simultaneous providers, or LM
Studio support.

The initial operational model is:

``` text
Lumen Servire
      │
      ▼
Lumen Praebere
      │
      ▼
    Ollama
      │
      ▼
Configured Qwen model
```

------------------------------------------------------------------------

## Component Boundary

Praebere owns knowledge of how the external model provider is operated.

For the initial Ollama implementation, Praebere is responsible for
translating Lumen-level operations such as:

``` text
status
start
load
unload
stop
```

into the appropriate Ollama operations.

Servire does **not** need to know how Ollama is started, how a model is
loaded or unloaded, or how provider state is queried.

Servire manages Praebere in the same way that it manages other Lumen
components.

------------------------------------------------------------------------

## Initial State Model

Praebere should expose sufficient state for Servire to present the
condition of both the provider and configured model.

A deliberately simple initial state model is:

``` text
Provider:
  stopped
  running

Model:
  unavailable
  available
  loaded
```

This allows important cases to be distinguished.

### Ollama not running

``` text
Provider: stopped
Model: unavailable
```

Praebere can offer the provider **Start** operation.

### Ollama already running

Praebere must detect an existing Ollama instance rather than assuming it
owns the provider process.

If the configured Qwen model is available:

``` text
Provider: running
Model: available
```

If it is already resident/loaded:

``` text
Provider: running
Model: loaded
```

### Model availability

A configured model being installed/available is distinct from it
currently being loaded.

Praebere should preserve this distinction where Ollama can report it
reliably.

------------------------------------------------------------------------

## Initial Operations

### Status

Report:

-   whether Ollama is running;
-   whether the configured Qwen model is available;
-   whether the configured model is currently loaded, where
    determinable;
-   enough diagnostic information for Servire validation and
    presentation.

### Start

Start Ollama when it is not already running.

If Ollama is already running, Praebere should recognise the existing
provider rather than starting a second instance.

Provider startup must remain independent from the Lumen Stack Start
operation.

### Load

Load or activate the configured Qwen model.

The exact Ollama mechanism should be encapsulated entirely inside
Praebere.

### Unload

Unload the configured Qwen model without necessarily stopping the Ollama
provider itself.

This distinction is important because provider lifecycle and model
lifecycle are separate concerns.

### Stop

Stop Ollama through an explicit Praebere operation.

Stopping the Lumen stack must **not** implicitly stop Ollama merely
because Lumen was using it.

------------------------------------------------------------------------

## Servire Integration

Once Praebere M1 exists, it will be integrated into Servire as a
**Managed Component of Lumen**.

Servire will expose Praebere's state and lifecycle controls without
implementing Ollama-specific behaviour itself.

Conceptually:

``` text
Servire
   │
   ├── Start Praebere
   ├── Stop Praebere
   ├── View provider/model status
   │
   └── Invoke provider/model operations
             │
             ▼
          Praebere
             │
             ▼
           Ollama
```

Praebere itself is a Lumen component, while Ollama remains an external
provider.

### Lumen Stack Start

Ollama should **not** be silently started as part of Lumen Stack Start.

Lumen requires a model provider, but the provider is not owned by the
Lumen stack lifecycle.

Servire validation should identify when the required model-provider
capability is unavailable before starting the stack.

This preserves the same general principle used elsewhere in Lumen:
external providers can exist independently of Lumen.

------------------------------------------------------------------------

## Initial Dependency Position

For the current full-stack deployment, the conceptual model path
remains:

``` text
Client
  ↓
Pontis
  ↓
Vestigare
  ↓
Repetere
  ↓
Moderari
  ↓
Model Provider
```

Praebere initially controls the provider but does not need to be
inserted into the conversational data path merely to provide lifecycle
control.

The exact later Moderari/Praebere runtime integration should be designed
as a separate milestone rather than being assumed by M1.

------------------------------------------------------------------------

## Future Rogare Integration

A later Praebere milestone will provide Rogare with model-provider
information.

The intended user flow is:

``` text
Rogare
   │
   ├── query available provider/model choices
   │
   ▼
Praebere
   │
   ├── providers
   ├── available models
   └── provider/model capabilities
```

Rogare can then allow the user to choose a model without embedding
Ollama- or LM-Studio-specific logic in the conversational client.

Conceptually:

``` text
Get providers
      ↓
Get available models
      ↓
User selects model
      ↓
Praebere activates model
      ↓
Conversation starts
```

This work is explicitly **out of scope for Praebere M1**.

------------------------------------------------------------------------

## Future Moderari Integration

Moderari currently knows the configured model-provider endpoint.

A future milestone should consider integrating Moderari with Praebere so
that Moderari depends on a Lumen model-provider abstraction rather than
directly on Ollama-specific configuration.

The desired separation is:

``` text
Moderari
    │
    ▼
Praebere / Lumen model-provider abstraction
    │
    ├── Ollama
    ├── LM Studio
    └── future providers
```

The exact runtime relationship requires design before implementation. In
particular, Praebere should not automatically become a model-traffic
proxy unless that provides a clear architectural benefit.

------------------------------------------------------------------------

## Future Provider Expansion

Praebere is intentionally designed so that Ollama is an implementation
adapter rather than the definition of the component.

Future providers may include:

``` text
Praebere
   │
   ├── Ollama adapter
   ├── LM Studio adapter
   └── future provider adapters
```

Provider capabilities may differ. Praebere should eventually expose
those capabilities rather than pretending every provider supports
identical lifecycle operations.

Potential future capabilities include:

-   provider discovery;
-   model discovery;
-   model metadata;
-   model selection;
-   model loading/unloading;
-   provider-specific health;
-   multiple configured providers;
-   local versus remote providers;
-   provider capability reporting.

These are not requirements for the initial implementation.

------------------------------------------------------------------------

## Relationship to Pontis

Praebere and Pontis occupy analogous architectural boundaries but have
different responsibilities.

``` text
External tool/client side                 External model side

Pi / other clients                        Ollama / LM Studio
       │                                         │
       ▼                                         ▼
    Pontis                                   Praebere
       │                                         │
       └──────────── Lumen components ───────────┘
```

Pontis remains responsible for communication/session bridging.

Praebere is responsible for model-provider abstraction and
provider/model lifecycle.

Neither component should absorb the other's responsibilities.

------------------------------------------------------------------------

## Relationship to Servire

Servire is the **Operational Control Plane**.

Praebere is the **Model Provider** component.

This distinction should remain explicit:

``` text
Servire decides WHAT operational action is requested.
Praebere knows HOW to perform it for the model provider.
```

This prevents Servire becoming coupled to Ollama and allows future
provider implementations without redesigning the Servire control plane.

------------------------------------------------------------------------

## Proposed M1 Configuration Direction

The exact schema should be finalised during implementation, but Praebere
will require configuration equivalent to:

``` yaml
provider:
  type: ollama
  endpoint: http://127.0.0.1:11434

model:
  name: qwen2.5-coder:14b-32k
```

Provider executable/lifecycle configuration can be added as required by
the actual Ollama control mechanism.

The configured model remains fixed for M1.

------------------------------------------------------------------------

## Praebere M1 Acceptance Criteria

The first Praebere milestone is complete when:

1.  Praebere runs as an independent Lumen component.
2.  Praebere has its own configuration.
3.  It detects whether Ollama is already running.
4.  It can start Ollama when required.
5.  It can determine whether the configured Qwen model is available.
6.  It can load the configured Qwen model.
7.  It can report whether the configured model is loaded where Ollama
    exposes that state.
8.  It can unload the configured model.
9.  It can explicitly stop Ollama.
10. Its API exposes provider/model status and lifecycle operations in
    provider-neutral Lumen terminology.
11. Servire can subsequently manage Praebere without containing
    Ollama-specific lifecycle code.
12. Lumen Stack Start/Stop remains separate from Ollama Start/Stop.

------------------------------------------------------------------------

## Development Sequence

The agreed sequence is:

``` text
1. Complete current Servire updates
2. Implement Lumen Praebere M1
3. Integrate Praebere into Servire as a Managed Component
4. Later: Rogare provider/model choice
5. Later: Moderari model-provider abstraction
6. Later: additional providers such as LM Studio
```

------------------------------------------------------------------------

## Architectural Decision

**Lumen Praebere is a first-class Lumen component responsible for
model-provider control and abstraction.**

Its first implementation will support only Ollama and the configured
Qwen model, providing start, load, unload, stop and status operations.

Ollama remains an external provider and its lifecycle remains
independent of the Lumen Stack Start/Stop lifecycle.

This establishes the provider boundary now without prematurely
implementing the broader model-discovery and provider-selection
capabilities that Praebere can support later.
