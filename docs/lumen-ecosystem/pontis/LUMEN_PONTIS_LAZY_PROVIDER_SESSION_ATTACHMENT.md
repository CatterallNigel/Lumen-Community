# Lumen Pontis — Lazy Provider Session Attachment

**Status:** Architectural design note  
**Date:** 2026-08-10  
**Component:** Lumen Pontis

## Purpose

Pontis is responsible for maintaining the relationship between downstream Lumen sessions and the northbound clients or providers that consume their traffic.

A client should not need to understand how a provider session is established. Likewise, components such as Lumen Replay and Lumen Rogare must not contain Pi- or ACP-specific session-management logic.

Pontis owns that responsibility.

This document defines **Lazy Provider Session Attachment**: the rule Pontis uses when traffic arrives for a session that does not yet have an appropriate northbound destination.

---

## Architectural Invariant

> **Pontis maintains the relationship between downstream Lumen sessions and northbound clients/providers. When northbound traffic arrives for a session with no attached northbound destination, Pontis establishes a provider session using that traffic as the initial interaction, associates the provider session with the existing Pontis session, and thereafter routes traffic normally.**

The current provider is **Pi**, and the current provider-session protocol is **ACP**.

Those are implementation details, not permanent architectural constraints.

Pontis must therefore be designed around the concepts of **provider** and **provider session**, rather than hard-coding the architectural model around Pi or ACP.

---

## Direction of Traffic

For clarity, Pontis treats traffic direction relative to the Lumen stack.

### Southbound

Southbound traffic travels from a client/provider through Pontis into the Lumen stack.

Typical flow:

```text
Client / Provider
        |
        v
      Pontis
        |
        v
      Trace
        |
        v
      Replay
        |
        v
      Lumen
        |
        v
      Model
```

When Pontis receives southbound traffic, it records or maintains the session association required to route subsequent traffic correctly.

### Northbound

Northbound traffic travels from the Lumen stack back toward the originating client/provider.

Typical flow:

```text
Model / Lumen stack
        |
        v
      Pontis
        |
        v
Client / Provider
```

Pontis examines the session associated with that traffic and determines its valid northbound destination.

If an existing destination is attached, Pontis routes the traffic normally.

If no northbound destination is attached, Pontis performs **Lazy Provider Session Attachment**.

---

## Lazy Provider Session Attachment

Provider sessions should not be created merely because a Lumen conversation exists.

Instead, Pontis creates a provider session only when traffic requires one.

The decision is:

```text
Northbound message arrives
          |
          v
Does Pontis have an attached
northbound destination for session?
       /             \
     yes              no
      |                |
      v                v
Route normally     Select provider
                       |
                       v
                Create provider session
                using this message
                       |
                       v
                Associate provider
                session with Pontis
                session
                       |
                       v
                Continue normal routing
```

This is deliberately broader than:

> Start ACP when a tool call is detected.

Pontis does not need to understand that a particular message is a tool call in order to decide whether a provider session is required.

The fundamental condition is:

> **Traffic exists for a session, but Pontis has no valid destination for that traffic.**

That is a routing/session-management problem, and therefore belongs to Pontis.

---

## Replay Behaviour

Replay must remain independent of ACP, Pi, and provider-session management.

Its responsibility at a fork is:

```text
Detect first divergence
        |
        v
Persist fork evidence
        |
        v
End comparison
        |
        v
Become transparent
        |
        v
Pass the fork-causing response onward unchanged
```

The critical invariant is:

> **Forking terminates replay comparison. It does not terminate traffic.**

Replay must not create an ACP session, invoke Pi, execute tools, or ask Pontis explicitly to create a provider session.

After the fork, Replay is a transparent proxy.

### UI-Initiated Replay

A Replay started from Servire may initially have no live Pi/provider session attached.

The private replay can therefore execute south of Pontis while comparison remains active.

If the replay forks:

```text
Model response
      |
      v
    Lumen
      |
      v
Replay detects fork
      |
      | comparison ends
      | response unchanged
      v
    Trace
      |
      v
    Pontis
```

Pontis receives northbound traffic for the replay session.

If that session has no attached northbound destination, Pontis applies Lazy Provider Session Attachment:

```text
Fork response reaches Pontis
           |
           v
No northbound destination
           |
           v
Create provider session
           |
           v
Deliver fork response to provider
           |
           v
Provider handles response normally
```

With the present implementation, Pontis creates a **Pi ACP session**.

If the divergent response contains a tool call, Pi can execute it.

The resulting conversation then continues normally through the stack.

Replay remains transparent throughout the post-fork conversation.

Trace remains recording until the conversation reaches its defined terminal condition.

---

## Rogare Behaviour

Rogare uses the same Pontis mechanism.

Rogare must not contain special logic that says:

> Start Pi ACP.

Rogare is simply a client of Pontis.

It sends conversational traffic through Pontis and receives responses through Pontis.

If a conversation subsequently produces northbound traffic for which Pontis has no attached provider destination, Pontis performs the same Lazy Provider Session Attachment used for Replay.

Therefore:

```text
Rogare
   |
   v
Pontis
   |
   v
Lumen stack
```

and, when required:

```text
Northbound traffic
       |
       v
     Pontis
       |
       | no attached provider
       v
Create provider session
       |
       v
       Pi
```

There is no Rogare-specific ACP architecture and no Replay-specific ACP architecture.

There is one Pontis provider-session architecture.

---

## Current Pi / ACP Implementation

Initially:

- the only provider is **Pi**;
- Pontis already knows how Pi is launched and managed;
- provider sessions are established using **ACP**;
- Pontis maintains the association between the Lumen/Pontis conversation session and the corresponding Pi ACP session.

Conceptually:

```text
Pontis Session
    |
    +-- downstream Lumen session
    |
    +-- northbound client, if present
    |
    +-- provider attachment
            |
            +-- provider: Pi
            +-- protocol: ACP
            +-- ACP session ID
```

Once the provider attachment exists, Pontis reuses it rather than creating another provider session for every message.

Normal HTTP traffic can continue through the existing Pontis/Lumen path; ACP exists to provide the provider-side session capability when it is required.

---

## Future Providers

The architecture must not assume that every future provider:

- is Pi;
- supports ACP;
- exposes tools in the same way;
- uses the same session lifecycle;
- uses the same tool names;
- or even requires an external provider session for the same reasons.

Future Pontis configuration may therefore describe multiple providers and their capabilities.

For example:

```text
Provider
  identity
  protocol
  capabilities
  session factory
  session lifecycle
  routing policy
```

Provider selection is a future concern.

The first implementation can select Pi unconditionally because Pi is presently the only provider.

The important requirement is that this assumption remains behind the Pontis provider abstraction rather than leaking into Replay, Rogare, Trace, Lumen, or other components.

---

## Responsibility Boundaries

### Pontis owns

- Pontis session identity and association.
- Northbound destination resolution.
- Provider selection.
- Lazy provider-session creation.
- Provider-session reuse.
- Pi ACP session management in the current implementation.
- Mapping between Pontis/Lumen sessions and provider sessions.
- Routing traffic to the appropriate attached destination.

### Replay owns

- Replay-plan execution.
- Behaviour comparison.
- Match/fork detection.
- Fork evidence.
- Transition to transparent proxy mode after the first divergence.

Replay does **not** own provider sessions.

### Rogare owns

- User console interaction.
- Sending user conversational traffic.
- Presenting conversational responses and appropriate operational feedback.

Rogare does **not** own provider sessions.

### Trace owns

- Recording traffic and behavioural evidence.
- Recording lifecycle.

Trace does **not** own provider routing.

### Lumen owns

- Model orchestration and its existing reasoning/continuity responsibilities.

Lumen does not need to know whether the northbound consumer is Rogare, Pi, another client, or a future provider.

---

## Related ACP Content Boundary

Lazy Provider Session Attachment also provides the correct architectural location for the previously observed ACP output contamination issue.

Pontis must distinguish between:

- genuine conversational/model content;
- tool interactions;
- provider protocol events;
- operational/status output;
- progress/heartbeat output;
- Lumen `\obt` informational output.

Operational or status output may be exposed to Rogare or Servire where useful, but it must not accidentally be promoted into assistant conversation history and subsequently returned to Lumen as model context.

This classification belongs at the Pontis provider/protocol boundary.

It should be implemented as part of the same Pontis/Rogare development rather than as a narrow special-case filter for the currently observed `\obt` symptom.

---

## Design Consequence

The important simplification is:

> **Clients produce and consume traffic. Pontis owns the problem of making sure that traffic has a valid session destination.**

Neither Replay nor Rogare needs to know when ACP is required.

Neither needs to know that Pi is currently the provider.

When Pontis encounters traffic that cannot be routed through an existing session association, Pontis establishes the required provider relationship and then resumes normal routing.

This keeps provider/session complexity inside Pontis and preserves the single-responsibility boundaries of the wider Lumen architecture.
