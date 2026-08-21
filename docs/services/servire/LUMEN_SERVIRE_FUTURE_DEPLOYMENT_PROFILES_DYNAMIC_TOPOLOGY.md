# Lumen Servire — Future Development: Deployment Profiles and Dynamic Topology

## Purpose

This note records a future Servire development requirement arising from the move toward a modular Lumen stack.

The current Servire configuration assumes the complete Lumen stack is present and therefore treats the configured dependency graph as fixed. That is appropriate for the current implementation and testing phase, but it will not be sufficient once Lumen is deployed in partial or user-selected configurations.

The future requirement is for Servire to understand the **active deployment topology**, rather than assuming that every Lumen component is always present.

## 1. Deployment Profiles

Servire should support named deployment profiles that define which Lumen components are enabled for a particular installation or runtime.

Example profiles might include:

```text
minimal
  Pontis
  Moderari
  Model Provider
  Persistence

observed
  Pontis
  Vestigare
  Moderari
  Model Provider
  Persistence

full
  Rogare
  Pontis
  Vestigare
  Repetere
  Moderari
  Model Provider
  Persistence
```

The exact profiles should remain configurable rather than being hard-coded.

A profile determines which components Servire manages, which dependencies are mandatory, the runtime request path through the stack, which interfaces are exposed, which health checks participate in Stack Health, and which components participate in Stack Start, Stop and Restart.

## 2. Conditional Dependency Graph

The dependencies currently shown in Servire should eventually be interpreted as dependencies for the **active deployment profile**, not immutable dependencies of the component.

For example, in the full stack the normal conversational path is:

```text
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
Model
```

In a minimal stack the path may instead be:

```text
Client
  ↓
Pontis
  ↓
Moderari
  ↓
Model
```

Servire should build and validate the dependency graph from the enabled components rather than requiring disabled components simply because they exist in the complete architecture.

## 3. Dynamic Topology Recalculation

A future extension should allow Servire to recalculate the active routing topology when optional components are deliberately stopped or disabled.

For example, if the full stack is running:

```text
Pontis → Vestigare → Repetere → Moderari
```

and **Vestigare is deliberately stopped**, then Repetere can no longer operate in the intended full-stack route because its recording/replay path depends upon Vestigare.

Servire could therefore perform a controlled topology transition:

```text
Before:

Pontis → Vestigare → Repetere → Moderari

After Vestigare is disabled:

Pontis ───────────────────────→ Moderari
```

In this state, Vestigare is stopped, Repetere is stopped or marked unavailable because its required recording path no longer exists, Pontis is reconfigured to route directly to Moderari, and the remaining stack continues operating as a reduced Lumen profile.

This should be treated as an **explicit topology change**, not simply as an arbitrary port substitution.

## 4. Routing Should Be Role-Based, Not Port-Based

Although the practical effect may be that Pontis changes its downstream port from Vestigare's port to Moderari's port, Servire should not model this as:

```text
11438 → 11436
```

The future abstraction should instead be:

```text
Pontis downstream role:
  recorder
```

becoming:

```text
Pontis downstream role:
  orchestrator
```

Servire can then resolve the actual endpoint from the active component configuration.

This avoids coupling topology logic to fixed ports and will allow port changes, remote components, containerised deployments, multiple stacks, and alternate implementations of the same role.

## 5. Controlled Component Shutdown

Stopping a component that participates in the active route should become a Servire workflow rather than a simple process termination.

Example: **Stop Vestigare**

Servire should determine that Vestigare participates in the active route, Repetere depends on the recording path, and Pontis currently routes through Vestigare.

A controlled transition could be:

```text
1. Stop accepting new work through the affected path.
2. Allow or cancel active work according to policy.
3. Stop Repetere if it cannot operate without Vestigare.
4. Stop Vestigare.
5. Reconfigure Pontis downstream to Moderari.
6. Verify Moderari is healthy.
7. Verify Pontis can reach the new downstream target.
8. Mark the active topology as reduced/degraded-by-design.
```

The reverse operation should also be possible when Vestigare/Repetere are re-enabled.

## 6. Degraded vs Intended Reduced State

Servire should distinguish between:

- **unexpected degradation** — a required component has failed;
- **intentional reduced topology** — the user deliberately disabled an optional component or selected a smaller deployment profile.

For example:

```text
Vestigare unexpectedly unavailable
  → DEGRADED / ERROR

Vestigare deliberately disabled in active profile
  → READY / REDUCED PROFILE
```

A deliberately minimal Lumen deployment should not appear unhealthy simply because Replay or Trace is absent.

## 7. Configuration Direction

A future configuration model could evolve toward something similar to:

```yaml
profiles:
  full:
    components:
      - rogare
      - pontis
      - vestigare
      - repetere
      - moderari

  minimal:
    components:
      - pontis
      - moderari

routing:
  full:
    pontis: vestigare
    vestigare: repetere
    repetere: moderari

  minimal:
    pontis: moderari
```

The exact schema should be designed when this work is scheduled. The important requirement is that **Servire owns topology selection and validation**, while each component continues to own its own local runtime configuration.

## 8. Architectural Principle

> **Servire should manage the topology of Lumen, not merely the processes that happen to make up one fixed topology.**

This allows Lumen to remain modular.

A user should be able to run:

```text
Pontis + Moderari
```

or:

```text
Pontis + Vestigare + Moderari
```

or:

```text
Rogare + Pontis + Vestigare + Repetere + Moderari
```

without requiring different component implementations.

Servire becomes responsible for assembling the selected services into a valid runtime graph.

## 9. Current Decision

This work is **not part of the current Servire update**.

For the present development pass, Servire continues to assume the complete stack, the current dependency graph remains valid, Stack Start/Stop continues to operate against the full configured stack, and component ports and routing remain statically configured.

The profile-aware and dynamic-topology work should be scheduled as a future Servire milestone once the current naming, configuration, Ollama lifecycle and log-management work is complete.

## Future Milestone Candidate

### Servire — Deployment Profiles & Dynamic Topology

Scope:

- named deployment profiles;
- conditional dependency graphs;
- role-based routing;
- topology-aware Start/Stop;
- controlled fallback routing;
- intentional reduced-stack health state;
- automatic dependency suppression where a component is not part of the active topology;
- restoration of the full route when disabled components are re-enabled.

This would allow Servire to evolve from a fixed-stack operational controller into a genuine **Lumen topology manager**.
