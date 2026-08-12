# Lumen Offline Clear Logs Management Command

**Status:** Architectural Standard\
**Reference implementation:** Rogare --- Console\
**Applies subsequently to:** Moderari --- Orchestrator, Servire ---
Service, Pontis --- Bridge, Vestigare --- Trace, Repetere --- Replay,
and future Lumen services.

## 1. Purpose

Every Lumen service shall provide a standard offline management command
for clearing its own logs.

Rogare is the first implementation and serves as the reference pattern
for the other services when they are revisited.

The command is intentionally **not an HTTP API**. Log deletion remains
the responsibility of the component that owns the logs, while Servire
may orchestrate the lifecycle around the operation.

## 2. Standard Command

For Rogare:

``` cmd
python -m lumen_rogare clear-logs
```

The equivalent convention for another component is:

``` text
python -m <lumen_component> clear-logs
```

Examples:

``` cmd
python -m lumen_moderari clear-logs
python -m lumen_pontis clear-logs
python -m lumen_vestigare clear-logs
python -m lumen_repetere clear-logs
```

The exact Python package name of an existing component must be respected
when the capability is retrofitted.

## 3. Log Location

A Lumen service owns logs only beneath:

``` text
$ROOT/logs/
```

`$ROOT` is the root directory of that service installation/repository.

The `clear-logs` command shall operate only on the owning service's
`$ROOT/logs` contents.

It shall not delete the `logs` directory itself.

## 4. Architectural Invariant

> **A component owns and understands its own logs. Servire may
> orchestrate the lifecycle operation, but it does not directly
> manipulate another component's log files.**

This preserves the wider Lumen invariant:

> **Each component has one clearly bounded responsibility.**

Servire therefore does not need knowledge of another component's log
paths, file naming conventions, container mounts, rotation scheme, or
filesystem implementation.

## 5. Offline-Only Requirement

`clear-logs` is an **offline management command**.

It is available for successful execution only when the owning service is
stopped.

Before deleting anything, the command must determine whether the service
is active.

Behaviour:

``` text
python -m <component> clear-logs
              |
              +-- Is the service active?
                    |
                    +-- YES -> refuse operation
                    |          delete nothing
                    |          report reason
                    |          exit non-zero
                    |
                    +-- NO  -> clear $ROOT/logs contents
                               report result
                               exit 0
```

The implementation must fail safely. If the command cannot reliably
establish that the service is stopped, it must not delete the logs.

## 6. Command Isolation

Executing `clear-logs` must **not start the normal service**.

In particular, the command must not unnecessarily initialise:

-   the HTTP/FastAPI application;
-   listeners or service ports;
-   downstream component connections;
-   provider connections;
-   ACP sessions;
-   conversational sessions;
-   model connections;
-   tracing or replay behaviour;
-   normal runtime background tasks.

Argument dispatch should occur sufficiently early in the module entry
point that the clear operation remains independent of normal application
startup.

For Rogare:

``` text
python -m lumen_rogare
        |
        +-- normal invocation ------> start Rogare
        |
        +-- clear-logs -------------> offline clear operation -> exit
```

## 7. Deletion Scope

The command may delete only entries owned by the component beneath:

``` text
$ROOT/logs/
```

It must not alter:

-   configuration;
-   conversation state;
-   session state;
-   checkpoints;
-   replay data;
-   trace data outside the component's own log directory;
-   caches;
-   databases;
-   user data;
-   another Lumen component's files;
-   service lifecycle configuration.

The operation is strictly a **log-clear operation**.

If `$ROOT/logs` is already empty, the operation is successful and should
exit with code `0`.

## 8. Safety Behaviour

The implementation shall follow these rules:

1.  Resolve the service root and log directory deterministically.
2.  Confirm the target is the owning component's `$ROOT/logs`.
3.  Confirm the service is not active.
4.  Do not follow deletion targets outside the owned log directory.
5.  Delete the contents of `$ROOT/logs`, not `$ROOT/logs` itself.
6.  On an unexpected filesystem or permission error, stop and return a
    non-zero exit code.
7.  Never start the service merely to perform the operation.
8.  Never partially broaden the deletion scope in response to an error.

The implementation should be idempotent: running `clear-logs` repeatedly
while the service is stopped and the log directory is empty remains a
successful operation.

## 9. Console Result

Successful execution should provide a concise human-readable result, for
example:

``` text
Rogare logs cleared successfully (4 files removed).
```

An already-empty directory may report:

``` text
Rogare logs cleared successfully (0 files removed).
```

If the service is active:

``` text
Rogare is running. Logs cannot be cleared while the service is active.
```

The active-service case must return a non-zero process exit code and
delete nothing.

## 10. Servire Integration

Servire may expose a **Clear Logs** action for a managed component, but
Servire does not perform the filesystem deletion itself.

The managed workflow is:

``` text
Servire
   |
   +-- Is component running?
   |       |
   |       +-- YES -> Stop component
   |                  confirm stopped
   |
   +-- invoke component's `clear-logs`
   |
   +-- inspect command result
   |
   +-- optionally restore previous running state
```

The precise Servire UI/workflow behaviour, including whether restart is
automatic or explicitly requested, is a Servire implementation concern.

The log deletion itself remains component-owned.

## 11. Reference Implementation: Rogare

Rogare shall be the first implementation of this standard.

Required command:

``` cmd
python -m lumen_rogare clear-logs
```

The Rogare implementation must:

-   recognise `clear-logs` before normal service startup;
-   verify that Rogare is stopped;
-   clear only `$ROGARE_ROOT/logs/*`;
-   preserve the `$ROGARE_ROOT/logs` directory;
-   perform no normal Rogare runtime initialisation;
-   report the number of removed entries/files as appropriate;
-   return `0` on success;
-   return non-zero when Rogare is active or the operation cannot be
    completed safely.

Tests should cover at minimum:

-   stopped service with populated logs;
-   stopped service with empty logs;
-   active service refusal;
-   preservation of the `logs` directory;
-   protection of files outside `logs`;
-   filesystem/deletion failure;
-   confirmation that normal service startup is not entered by
    `clear-logs`.

## 12. Rollout

After Rogare establishes and validates the implementation, the same
behavioural contract shall be applied to the other Lumen services during
their documented post-Rogare work.

The implementation may differ internally where a component's runtime
requires it, but the externally observable command and safety semantics
should remain consistent.

## 13. Standard Summary

The Lumen standard is:

> **`python -m <lumen_component> clear-logs` is an offline,
> component-owned management command. It clears only that component's
> `$ROOT/logs` contents, refuses to operate unless the service is
> confirmed stopped, performs no normal service initialisation, and
> exits immediately when complete.**

This command is the canonical mechanism by which Servire will request
log clearing across independently managed Lumen components.
