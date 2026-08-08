# Milestone 7 Local Configuration Merge

Milestone 7 adds component workspace metadata to Servire's configuration schema.

**Do not replace an existing installation-specific `config.yml`.** Keep local paths,
credentials, ports, model selections, MongoDB URI, and other site-specific settings.

Update the service version to:

```yaml
service:
  version: 0.7.0
```

Then add these interface sections to the corresponding existing service definitions.

## Lumen

```yaml
interfaces:
  - key: operations
    display_name: Lumen Operations
    url: http://127.0.0.1:11435/operations
    availability_service_key: trace
  - key: checkpoints
    display_name: Lumen Checkpoints
    url: http://127.0.0.1:11435/checkpoints
    availability_service_key: trace
```

The availability owner is Trace because the current Trace runtime transparently proxies
these Lumen interfaces.

## Lumen Replay

```yaml
interfaces:
  - key: operator
    display_name: Replay
    url: http://127.0.0.1:11437/
```

## Lumen Trace

```yaml
interfaces:
  - key: trace
    display_name: Trace
    url: http://127.0.0.1:11435/trace
```

The URLs are installation configuration. If ports or routes change, only `config.yml`
needs to change; Servire has no hard-coded component UI addresses.
