# Servire Component Vocabulary and Configuration Standardisation

**Version:** 0.8.5  
**Status:** Implemented

Servire now uses one canonical Lumen family vocabulary throughout the live workspace and resolved configuration presentation. Latin names identify components; the English label states the component responsibility.

## Canonical component vocabulary

- Servire — Service
- Moderari — Orchestrator
- Repetere — Replay
- Vestigare — Recorder
- Pontis — Communication Bridge
- Rogare — Conversational Client

Capability names remain capability names inside a component workspace. For example, Repetere can expose a **Replay** interface, Vestigare can expose a **Trace** interface, and Rogare can expose a **Console** interface. This prevents a capability label from being confused with the component's responsibility.

## Configuration conventions

The managed service keys remain the stable Latin component identifiers: `moderari`, `repetere`, `vestigare`, `pontis`, and `rogare`. Runtime Python package names remain implementation details (`lumen_moderari`, `lumen_repetere`, `lumen_vestigare`, `lumen_pontis`, and `lumen_rogare`).

Servire's configuration now declares component configuration files consistently where they are available for inspection, including Pontis `config.yml`. The stale automatic Moderari `--clear-logs` launch argument has been removed from the example configuration; log maintenance is a separate component-owned operation rather than an implicit side effect of normal startup.

## UI conventions

Top-level workspace navigation uses the canonical `Latin name — responsibility` form. Service cards and configuration panels use the same responsibility labels from the domain model rather than maintaining separate UI-only mappings. Internal dependency roles are rendered through that same vocabulary.

The dashboard heading **Managed Services** is now **Managed Components**. External systems such as Ollama and MongoDB remain **External Dependencies**.

Operational log source keys and styling now follow the canonical component keys (`moderari`, `repetere`, `vestigare`, `pontis`, `rogare`, `servire`).
