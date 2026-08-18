# Lumen Pontis Milestone 5.1 Acceptance — Automatic Northbound Provider Attachment

## Architectural invariant

Replay has no knowledge of Pontis or ACP. On a fork Replay records the divergence, ends
comparison, becomes transparent, and lets the fork-causing response continue northbound.
Trace remains the next component in the stack and forwards that northbound traffic to Pontis.

Pontis alone decides whether a provider session is required.

## Northbound ingress

Pontis now exposes the internal northbound traffic boundary:

```text
POST /_pontis/northbound/{session_id}
```

A northbound message arriving for a session with no provider binding causes Pontis to:

1. resolve the provider working directory;
2. create a Pi ACP session lazily;
3. bind the ACP session to the Lumen/Pontis session;
4. deliver the same northbound message as the first provider interaction;
5. reuse that provider binding for subsequent northbound deliveries.

If a binding already exists, no new ACP session is created.

## Working directory

Pi requires a working directory when an ACP session is created. M5.1 moves the fallback for
that requirement into Pontis configuration:

```yaml
acp:
  default_cwd: "."
```

An upstream northbound delivery may provide a more specific `cwd`; otherwise Pontis uses the
configured provider default. This remains provider/session infrastructure and does not leak
Pi/ACP requirements into Replay.

## Existing provider delivery API

The explicit M5 endpoint remains available for diagnostics and future clients:

```text
POST /_pontis/sessions/{session_id}/deliver
```

It now uses the same configured `default_cwd` fallback.

## Traffic flow

The target fork flow is:

```text
Model
  -> Lumen
  -> Replay (detect fork, persist evidence, become transparent)
  -> Trace
  -> Pontis northbound ingress
  -> lazy Pi ACP session attachment when required
  -> Pi
```

Pi's subsequent model-facing traffic returns through the ordinary HTTP stack:

```text
Pi -> Pontis -> Trace -> Replay (transparent) -> Lumen -> Model
```

Replay must not call Pontis directly and Trace must not make provider-selection decisions.

## Integration boundary

M5.1 implements the Pontis side of this contract. For a UI/private Replay fork to exercise it,
Trace must forward the fork-causing northbound message, with its session identity, to Pontis'
northbound ingress. Pontis cannot attach a provider to traffic that never reaches Pontis.
