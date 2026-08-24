# Lumen Servire — Nuntius Control-Plane Configuration

**Status:** Proposed M0.1 Development  
**Service:** Servire  
**Related Service:** Nuntius

## 1. Purpose

Servire is already authoritative for the operational Lumen service topology.

M0.1 extends that responsibility so Servire can also describe the control-plane metadata Nuntius requires to route `\obt` commands and their responses.

Servire does not execute Nuntius routing.

It defines the active routing configuration.

> **Servire defines the active Lumen control-plane topology; Nuntius materialises and executes that topology at runtime.**

## 2. Why the Configuration Belongs in Servire

Servire already knows:

- which services are configured;
- which services are running;
- where they are running;
- when a service starts;
- when a service stops.

The same service definition can therefore describe:

- whether the service participates in `\obt`;
- which commands it owns;
- which successful query responses should also be delivered to other Lumen services.

This avoids placing domain policy inside Nuntius.

## 3. Service Configuration

Each service definition may include:

```text
obt_enabled
commands
```

Conceptually:

```yaml
service: praebere
endpoint: http://praebere:11431

obt_enabled: true

commands:
  providers:
    response_targets:
      - originator
      - rogare

  models:
    response_targets:
      - originator
      - rogare

  model_select:
    response_targets:
      - originator
```

The exact configuration syntax should follow existing Servire configuration conventions.

## 4. `obt_enabled`

`obt_enabled` identifies whether the service participates in the `\obt` control plane.

```text
obt_enabled: false
```

means that the service has no active `\obt` execution responsibilities.

A common service control endpoint may immediately return:

```text
204 No Content
```

when `obt_enabled` is false.

When true, the service may advertise one or more owned commands.

## 5. Command Ownership

Servire configuration defines which service owns a command.

Conceptually:

```text
replay list -> Repetere
models      -> Praebere
providers   -> Praebere
```

Nuntius uses this metadata to route directly to the owner rather than broadcasting the command to every running service.

Command ownership should be unambiguous in the active topology.

If two active service definitions claim the same exclusive command, that should be treated as a configuration error rather than silently resolved by Nuntius.

## 6. Response Targets

A query response may be required by:

- only the originating external/session client;
- the originator plus one or more Lumen services.

Servire defines this explicitly.

Example:

```yaml
models:
  response_targets:
    - originator
    - rogare
```

When Praebere returns:

```text
200 + model list
```

Nuntius:

1. returns the result to Pontis for the originating client;
2. sends the same result to Rogare.

This does not make Nuntius the owner of model state.

Praebere remains authoritative.

## 7. Originator

`originator` is a logical response target.

It does not identify a service endpoint.

For an external client, Nuntius returns the result to Pontis with the request/session correlation. Pontis then returns the result to the correct external client.

For an internal Lumen originator, the same correlation principle may be used without changing command ownership.

## 8. Bootstrap Catalogue

When Nuntius starts, it sends Servire a direct:

```text
\obt services
```

Servire's response should include the active information Nuntius requires to build its routing dictionary.

Conceptually:

```yaml
services:
  - name: repetere
    endpoint: http://repetere:11437
    obt_enabled: true
    commands:
      replay_list:
        response_targets:
          - originator

  - name: praebere
    endpoint: http://praebere:11431
    obt_enabled: true
    commands:
      models:
        response_targets:
          - originator
          - rogare
```

An empty `services` collection is valid.

## 9. Runtime Updates

When Servire starts or stops a service, it must update Nuntius.

The update travels through the normal Lumen control path:

```text
Servire
   |
   | \obt service available / unavailable
   v
Pontis
   |
   v
Nuntius
```

A service-available update must contain enough control-plane metadata for Nuntius to add its routes.

A service-unavailable update must allow Nuntius to remove the associated routes.

Nuntius may request a complete catalogue rebuild from Servire at any time.

## 10. Response Semantics

Servire configuration does not redefine the common service response contract.

The contract remains:

```text
204
    command not handled

200, no body
    execution succeeded

200 + body
    query succeeded; body is authoritative result

4xx / 5xx
    recognised command failed
```

`response_targets` apply to successful responses where there is a result to distribute.

## 11. M0.1 Logging Boundary

Do not add operational-log classification such as:

```text
log_class: user
```

to this initial configuration.

The need is recognised, particularly because internal Rogare polling and maintenance activity can make the operational log difficult to read.

However, filtering the operational log so the default view emphasises user-session activity is a separate Servire/Rogare development item.

When that work is undertaken, command metadata may be extended with a logging classification if appropriate.

It is explicitly not required for the first Nuntius configuration implementation.

## 12. Validation

Servire tests should verify:

- `obt_enabled` can be configured per service;
- active command ownership is emitted in `\obt services`;
- response targets are emitted correctly;
- an empty active catalogue is valid;
- service start adds the appropriate control-plane metadata;
- service stop removes it;
- duplicate exclusive command ownership is rejected or clearly reported;
- Nuntius can rebuild its routing dictionary entirely from Servire's authoritative catalogue.

## 13. Architectural Boundary

Servire defines configuration and operational truth.

Nuntius performs runtime routing.

Domain services execute their commands.

Pontis owns external session/client return routing.

This separation should remain explicit as the control plane evolves.
