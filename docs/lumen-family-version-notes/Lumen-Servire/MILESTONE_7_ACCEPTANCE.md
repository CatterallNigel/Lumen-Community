# Milestone 7 Acceptance — Unified Component Workspace

## Objective

Provide a single browser window for operating Lumen++ without duplicating the native
Lumen, Replay, or Trace interfaces inside Servire.

Servire remains the control plane. Component applications continue to own their own UIs
and behaviour.

## Delivered

- Persistent workspace navigation across Servire and configured component interfaces.
- Config-driven interface metadata; no UI endpoint is hard-coded in Servire.
- Multiple interfaces per component.
- Embedded native interfaces using iframes.
- Health-aware workspace availability.
- Optional availability ownership for proxied interfaces.
- Direct-open fallback for every embedded interface.
- Graceful unavailable presentation instead of a broken iframe.
- Read-only display of configured interface URLs in Servire's Configuration panel.
- No reverse proxying and no duplication of Trace/Replay/Lumen controls.

## Initial Lumen++ Workspace

The example configuration exposes:

- Servire
- Lumen Operations
- Lumen Checkpoints
- Replay
- Trace

Lumen Operations and Lumen Checkpoints are currently served through Trace's transparent
proxy, so their interface definitions use Trace as the availability owner while retaining
Lumen as the architectural component.

## Configuration Schema Addition

M7 adds an optional `interfaces` collection to each service:

```yaml
interfaces:
  - key: operations
    display_name: Lumen Operations
    url: http://127.0.0.1:11435/operations
    availability_service_key: trace
```

`availability_service_key` is optional. If omitted, the interface is considered available
when its owning service is available.

This is a schema addition. Existing installation-specific `config.yml` files should be
merged with the required `interfaces` entries rather than replaced. `config.example.yml`
is provided as a reference.

## Acceptance Criteria

- Dashboard navigation is generated from configured interfaces.
- Servire remains available as the first workspace tab.
- A configured available interface renders its native URL in an iframe.
- An unavailable interface presents a clear message and does not render a broken iframe.
- Every component workspace offers **Open directly** as a fallback.
- Multiple interfaces belonging to one component are supported.
- Interface availability may be delegated to another configured service for proxy cases.
- Unknown workspace routes return HTTP 404.
- Duplicate interface keys within one service are rejected.
- Invalid interface availability-service references are rejected at configuration load.
- Existing M1–M6 behaviour remains intact.
- pytest coverage remains at or above 95%.
- Ruff and mypy remain clean in the authoritative local Python 3.12.10 environment.

## Explicitly Deferred

M7 does not:

- reverse proxy component UIs through Servire;
- copy or recreate Trace, Replay, or Lumen controls;
- add Assess integration;
- edit component configuration;
- implement the post-roadmap clickable configuration actions;
- implement runtime client/model selection.
