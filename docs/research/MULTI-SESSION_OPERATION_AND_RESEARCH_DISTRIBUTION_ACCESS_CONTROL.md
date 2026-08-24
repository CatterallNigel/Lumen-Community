# Multi-Session Operation and Research Distribution Access Control

**Status:** Development Requirement  
**M0.1:** Concurrency validation required  
**General Research Distribution:** Single-active-operator enforcement required

## 1. Architectural Requirement — Concurrent Sessions

The Lumen stack should not depend upon there being only one active Lumen session.

Multiple simultaneous sessions must be tested to establish that session state is correctly isolated throughout the stack.

A representative test should include multiple concurrent sessions:

```text
Session A -> Moderari -> Model
          -> Trace A

Session B -> Moderari -> Model
          -> Trace B

Session C -> Moderari -> Model
          -> Trace C
```

Testing should verify that execution state, Trace evidence, prompt state, tool state and other session-specific information cannot cross session boundaries.

This is an architectural validation requirement for M0.1.

A restriction imposed upon the Research Distribution must not be used to conceal or avoid testing the underlying multi-session architecture.

## 2. Research Distribution Requirement — Single Active Operator

Although the underlying Lumen architecture should support multiple sessions, the general Research Distribution should initially permit only **one active Servire operator session per installation**.

This is an access and distribution constraint rather than a limitation of Lumen's internal architecture.

Importantly, the restriction should not bind Servire to localhost.

A legitimate installation may have the Lumen stack and model provider running on one machine while the researcher operates Servire remotely from another machine on the network.

For example:

```text
Lumen Host
    Ollama
    Praebere
    Moderari
    Trace
    Replay
    Servire
        |
        | network
        |
Researcher's Laptop
    Browser -> Servire
```

This must remain supported.

## 3. Behaviour When Another Operator Connects

Servire should recognise when an active operator session already exists.

A second browser or user attempting to establish another Servire session should not receive an obscure HTTP or application error.

Servire should display a clear explanatory page such as:

> **Lumen is currently in use**
>
> This installation already has an active Servire session.
>
> Close or exit the existing session before starting another.

The second connection must not disturb the existing operator session.

## 4. Operator Lease

The active operator should preferably be represented by a renewable lease rather than relying exclusively upon browser connection state.

Conceptually:

```text
Servire Operator Lease

session_id
acquired_at
last_heartbeat
expires_at
```

The active Servire UI periodically renews the lease.

While the lease remains valid, additional operator sessions are declined.

This prevents an abandoned browser, network interruption, crashed client or suspended laptop from permanently locking the installation.

If the existing operator disappears and the lease is no longer renewed, it expires and another operator may subsequently connect.

## 5. Explicit Session Release

Servire should provide an explicit mechanism such as:

**Exit / Release Session**

Normal termination should immediately release the operator lease rather than requiring the lease timeout to expire.

The timeout remains the recovery mechanism for abnormal termination.

## 6. Separation of Concerns

Two concepts must remain explicitly separate:

### Lumen Session Concurrency

Whether the underlying Lumen services can safely process multiple simultaneous isolated sessions.

This must be tested.

### Servire Operator Concurrency

How many human operators the distribution permits to use a particular installation simultaneously.

This is a distribution/licensing policy.

A single-active-operator Research Distribution therefore does **not** imply that Lumen itself is a single-session architecture.

## 7. Future Commercial Licensing

The same mechanism could later support commercial entitlement rather than requiring a different architecture.

Conceptually:

```text
Research Distribution:
    max_active_operators = 1

Commercial Licence:
    max_active_operators = N
```

The exact commercial licensing model remains undecided.

The important architectural principle is that licensing restrictions should be enforced at an appropriate access boundary rather than deliberately reducing the underlying capabilities of Lumen.

## Release Position

### M0.1

Required:

- validate multiple concurrent Lumen sessions;
- verify isolation between sessions.

Not necessarily required:

- Servire operator lease enforcement.

### General Research Distribution

Required before unrestricted research distribution:

- single-active-operator Servire policy;
- graceful rejection of additional operators;
- operator lease/timeout;
- explicit session release;
- remote operation from another machine remains supported.

This provides a practical research-distribution boundary without artificially constraining Lumen's internal architecture.