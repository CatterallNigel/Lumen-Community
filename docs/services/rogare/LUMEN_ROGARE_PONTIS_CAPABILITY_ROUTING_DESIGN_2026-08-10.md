# Lumen Rogare — Pontis Capability Routing Design

**Status:** Deferred design / future implementation
**Date:** 2026-08-09
**Purpose:** Capture the architectural discussion around introducing **Lumen Rogare** as a separate console/client and using **Pontis** to broker tool capabilities supplied by Pi. This work is intended to be picked up after Pontis has been integrated into the Lumen stack and the Servire/port changes have been completed.

---

## 1. Context

The Lumen service console should not be embedded directly into Servire. Instead, it should become a separate first-class Lumen module named **Rogare**.

Rogare is the human-facing interaction client for the Lumen stack. It should be exposed through Servire in the same manner as the other independently owned services/modules, rather than Servire itself becoming the conversational client.

The intended family of responsibilities is therefore:

- **Lumen** — behavioural and continuity processing.
- **Pontis** — protocol bridge and capability routing.
- **Trace** — observation and recording.
- **Replay** — behavioural reproduction/replay.
- **Assess** — behavioural assessment.
- **Rogare** — interaction/client console.
- **Servire** — service orchestration and operational management.

Each component should retain a single responsibility.

---

## 2. Rogare Architectural Invariant

> **Rogare is a client of the Lumen stack, not part of the Lumen processing path. It must interact with Lumen through Pontis using the same externally supported interfaces available to another client.**

Rogare must not have a privileged internal route into Lumen.

Conceptually:

```text
┌──────────────┐
│ Lumen Rogare │
│   Console    │
└──────┬───────┘
       │
       │ client request
       ▼
┌──────────────┐
│    Pontis    │
│ Protocol /   │
│ Capability   │
│    Bridge    │
└──────┬───────┘
       │
       ▼
   Lumen stack
```

This is useful architecturally and operationally because using Rogare also exercises the real external client path rather than an internal shortcut.

---

## 3. Rogare Does Not Own Tools

Rogare should initially declare and implement **no tools of its own**.

Its responsibility is deliberately narrow:

```text
send user input
receive conversational output
```

Pi remains the current provider of tools/capabilities.

This separates two roles that can otherwise appear coupled:

- **Rogare = interaction client**
- **Pi = capability/tool provider**

A Rogare-originated conversation may still use Pi's tools. Rogare itself does not need to know how those tools are implemented or execute them.

---

## 4. Pontis as the Tool Interception and Routing Point

When Rogare initiates a conversation, Pontis is responsible for arranging access to the capability provider and preserving the routing information required for subsequent tool calls.

The intended flow is:

```text
Rogare
   │
   │ user prompt
   ▼
Pontis
   │
   │ prompt + provider capabilities
   ▼
Lumen
   │
   ▼
Qwen
   │
   │ tool call required
   ▼
Lumen
   │
   ▼
Pontis              ← intercept/routing boundary
   │
   │ ACP tool invocation
   ▼
Pi                  ← tool/capability provider
   │
   │ ACP tool result
   ▼
Pontis
   │
   │ translated continuation
   ▼
Lumen
   │
   ▼
Qwen
   │
   │ conversational response
   ▼
Lumen
   │
   ▼
Pontis
   │
   ▼
Rogare
```

The important distinction is:

> **Rogare asks. Pi provides capabilities. Pontis brokers between them.**

Pontis must not send the Qwen tool request to Rogare merely because Rogare originated the user prompt. It knows that the requested capability was supplied by Pi and routes the invocation back to Pi.

Only the resulting conversational output is returned to Rogare.

---

## 5. ACP Session Creation for Rogare Conversations

Because Rogare does not itself provide tools, Pontis needs an ACP relationship with Pi in order to discover the capabilities that can be exposed for the Rogare conversation.

The proposed behaviour is **lazy ACP conversation/session creation**.

On the first Rogare message:

1. Rogare sends its first user prompt to Pontis.
2. Pontis determines that no Pi ACP session is currently bound to this Rogare conversation.
3. Pontis establishes the required ACP session with Pi.
4. Pi advertises its available tools/capabilities through ACP.
5. Pontis records those capabilities and their provenance.
6. Pontis binds the Rogare conversation to the Pi ACP provider session.
7. Pontis sends the request through Lumen to Qwen with the appropriate tool definitions available.

Conceptually:

```text
Rogare
   │
   │ first prompt
   ▼
Pontis
   │
   ├── no provider session bound
   │
   ├── ACP session/init ─────────────► Pi
   │                                  │
   │   ◄── capabilities / tools ──────┘
   │
   ├── bind:
   │      Rogare conversation R1
   │               ↕
   │      Pi ACP session P1
   │
   ▼
Lumen
   │
   │ prompt + Pi tool definitions
   ▼
Qwen
```

Pontis may independently verify that Pi is reachable as part of service startup/readiness. That is distinct from creating the ACP conversation/session associated with an individual Rogare conversation.

Therefore:

```text
Pontis startup/readiness:
    Is Pi available?

Rogare first prompt:
    Create/bind the Pi ACP conversation required for this client conversation.
```

---

## 6. Session Isolation

The initial design should favour a separate provider session for each Rogare conversation rather than treating Pi as one global conversational session.

```text
Rogare conversation A ─── Pontis ─── Pi ACP session A
Rogare conversation B ─── Pontis ─── Pi ACP session B
Rogare conversation C ─── Pontis ─── Pi ACP session C
```

This keeps conversation-specific state and provenance isolated, including matters such as:

- tool state;
- working context;
- permissions;
- tool results;
- conversation lifecycle;
- capability provenance.

The exact ACP session semantics should be confirmed against the implemented Pontis/Pi behaviour when this work begins.

---

## 7. Tools Belong to Capability Providers, Not Clients

A key conclusion from this design is that tool ownership should not be tied to the client that initiated the conversation.

Instead:

> **Tools belong to a capability provider, and Pontis maintains the mapping between advertised capabilities and their provider/session.**

A Pontis session may therefore conceptually contain:

```text
Client conversation
────────────────────────────────
client: Rogare
response destination: Rogare

model path:
    Pontis → Lumen → Qwen

capability providers:
    Pi ACP session P1
        ├── read_file
        ├── write_file
        ├── terminal
        └── ...
```

The client and capability provider are intentionally independent concepts.

---

## 8. Tool Names Are Not Portable Across Providers

Equivalent tools should **not** be assumed to have identical names across vendors, protocols or agent runtimes.

The same conceptual operation could be advertised as, for example:

```text
Pi             read_file
Provider A     file_read
Provider B     filesystem.read
Provider C     read
MCP server     read_text_file
```

Even when two providers advertise the same name, their schemas may differ.

For example:

```text
Provider A:
    read_file(path="src/main.py")

Provider B:
    read_file({
        "file_path": "src/main.py",
        "encoding": "utf-8"
    })
```

Therefore Pontis must never infer tool ownership or routing solely from the semantic meaning of a name such as `read_file`.

---

## 9. Capability Provenance Registry

When Pontis establishes the ACP session with Pi and receives its advertised tools, it should retain the relationship between each advertised capability and its provider session.

Conceptually:

```text
Pontis capability registry

provider_session: pi-123
protocol: ACP

advertised capabilities:
    read_file       → pi-123
    write_file      → pi-123
    list_directory  → pi-123
```

When those capabilities are exposed to Qwen, Pontis retains their provenance.

If Qwen subsequently requests `read_file`, Pontis does not merely search for something called `read_file`. It resolves the invocation to the provider capability that was advertised into that model interaction and routes it to the corresponding provider session.

This leads to a further Pontis invariant:

> **Pontis preserves capability provenance across protocol boundaries.**

---

## 10. Internal Capability Identity

Tool names alone are insufficient as durable capability identifiers because multiple providers may expose identical names.

Pontis should therefore eventually maintain an internal identity stronger than the model-visible tool name.

Conceptually:

```text
capability_id:
    provider: pi
    provider_session: abc123
    protocol: ACP
    native_name: read_file

model_visible_name:
    read_file
```

The internal identity can retain other metadata as required, such as the advertised argument schema and provider-specific invocation information.

This allows Pontis to distinguish between two independently advertised tools even when their native names are identical.

---

## 11. Name Collision Handling

Future Pontis configurations may have more than one capability provider:

```text
                    Pontis
                      │
          ┌───────────┼────────────┐
          ▼           ▼            ▼
         Pi       MCP Provider   Provider X
        ACP           MCP          ...
         │             │
    read_file     read_file
    terminal      search_files
```

If two capabilities would otherwise have the same model-visible name, Pontis must not guess which one the model intended.

A future capability exposure layer may need to:

- namespace capabilities;
- create model-visible aliases;
- retain a mapping from alias to provider-native identity;
- reject ambiguous registrations where safe disambiguation is impossible.

The exact naming/alias policy is deferred until multiple capability providers are required.

---

## 12. Important Safety/Isolation Rule

Pontis must never select an arbitrary available provider simply because a model emits a tool-like request.

The following behaviour would be incorrect:

```text
Rogare prompt
    ↓
Qwen requests a filesystem operation
    ↓
Pontis notices that Pi happens to have a filesystem tool
    ↓
Pi executes it
```

unless that Pi capability was explicitly advertised and associated with the active model interaction/session.

The correct rule is:

> **A model tool invocation may only be routed to a capability that Pontis explicitly advertised for that interaction and whose provider provenance Pontis retained.**

This prevents accidental capability leakage between sessions or providers.

---

## 13. Generalised Future Architecture

Although Pi is the current provider, the architecture should not make Pi a permanent special case.

The longer-term model is:

```text
                       ┌── Pi / ACP
                       │
Rogare ───► Pontis ────┼── MCP provider
                       │
                       ├── agent runtime
                       │
                       └── other capability provider

                │
                ▼
              Lumen
                │
                ▼
              Model
```

Pontis becomes responsible for preserving the relationship between:

- client conversation;
- response destination;
- model interaction;
- capability definitions exposed to the model;
- capability provider;
- provider protocol;
- provider session;
- tool invocation;
- returned tool result.

This does **not** mean Pontis becomes an agent or makes behavioural decisions. Its responsibility remains deterministic protocol/capability mediation and routing.

---

## 14. Servire Relationship

Rogare should be independently owned and versioned, while Servire exposes and manages it operationally in the same style as the other Lumen services.

Conceptually:

```text
Lumen Servire
│
├── Lumen
│   ├── status
│   ├── start/stop
│   └── health
│
├── Pontis
│   ├── status
│   ├── start/stop
│   └── health
│
├── Trace
│   ├── status
│   ├── start/stop
│   └── open UI
│
├── Replay
│   ├── status
│   ├── start/stop
│   └── open UI
│
└── Rogare
    ├── status
    ├── start/stop
    └── open console
```

Assess can similarly remain separately owned and exposed/managed according to its eventual operational requirements.

Servire must not absorb Rogare's client responsibilities.

---

## 15. Rogare-Controlled Trace Recording

Because Rogare is the human-facing interaction client and owns the conversational console, it should provide explicit controls that allow the user to start and stop Trace recording for the active Rogare conversation.

This does **not** transfer recording responsibility into Rogare. The responsibility boundary remains:

- **Rogare** — owns the user's intent to start or stop recording and presents the current recording state.
- **Trace** — owns recording state, recording lifecycle, captured evidence and storage.
- **Pontis/session infrastructure** — provides the conversation/session identity and correlation required to bind the recording to the correct live conversation.
- **Servire** — continues to manage Trace and Rogare operationally as independently owned services.

The architectural requirement is:

> **Rogare shall provide explicit user controls for starting and stopping Trace recording for the active conversation. Rogare owns only the user interaction and presentation of recording state; Trace remains authoritative for recording lifecycle and storage. Starting or stopping Trace must not alter the lifecycle of the underlying Rogare/Pontis conversation.**

Conceptually, the Rogare console may expose recording state as:

```text
Trace: ○ Off                  [Start Trace]
```

and, while recording:

```text
Trace: ● Recording — <trace>  [Stop Trace]
```

When the user starts recording, Rogare already has the identity of the active conversation. The user should therefore not normally need to select which session is to be recorded. The request should be correlated with the active Rogare/Pontis conversation and passed to Trace through the appropriate service interface.

Stopping Trace must stop evidence capture only. It must not terminate, reset, pause or otherwise alter the Rogare conversation.

This creates two intentional entry points into the same Trace lifecycle:

1. **Explicit/manual tracing from Rogare** — the user decides that the current live conversation should be recorded.
2. **Workflow-driven tracing** — Replay, Fiducia or another authorised Lumen workflow starts Trace because evidence capture is required.

Both paths should converge on the same Trace-owned recording API and lifecycle. Rogare must not implement a separate recording mechanism.

A further invariant therefore applies:

> **The initiator of a Trace recording may vary, but ownership of recording behaviour does not: Trace is always authoritative for recording lifecycle.**

---

## 16. Deferred Implementation Questions

The following should be resolved when Rogare development begins and against the actual Pontis/ACP implementation then in place:

1. Exact ACP session lifecycle used between Pontis and Pi for a Rogare conversation.
2. Whether Pi capabilities are discovered per ACP session or may be cached safely between sessions.
3. How capability changes during a live provider session are represented.
4. Exact capability registry structure inside Pontis.
5. How Pontis associates model-visible tool calls with internal capability identities.
6. Name-collision and aliasing rules when multiple providers expose the same native tool name.
7. Behaviour when a provider disappears during an active tool invocation.
8. Behaviour when the model emits a tool call that was not advertised for that interaction.
9. Conversation/session cleanup between Rogare, Pontis and Pi.
10. How Trace should represent capability-provider routing and tool provenance for later Replay/Assess use.
11. Whether Rogare should eventually display tool activity as read-only operational/conversational information while continuing to own no tool execution.
12. Exact service/API path Rogare should use to request Trace start/stop for its active conversation.
13. How Rogare obtains and reflects Trace's authoritative recording state, including failure or externally initiated state changes.
14. Trace naming/metadata conventions for manually initiated Rogare recordings.

---

## 17. Implementation Timing

This design is intentionally **not** the immediate development task.

Before beginning Rogare/capability-routing work:

1. Complete the current Pontis integration work.
2. Complete the required Lumen++ port adjustments.
3. Integrate Pontis correctly into Servire's service lifecycle and dependency model.
4. Verify the normal client → Pontis → Lumen path.
5. Verify ACP communication between Pontis and Pi.
6. Then revisit this document and design/implement Rogare against the proven Pontis architecture.

---

## 18. Core Architectural Invariants

For reference, the main conclusions are:

> **Rogare is a client of the Lumen stack, not part of the Lumen processing path.**

> **Rogare owns interaction, not tools.**

> **Pi is currently a capability/tool provider, independently of which client initiated the conversation.**

> **Pontis establishes and maintains the provider session required to make Pi capabilities available to a Rogare conversation.**

> **Tools belong to capability providers, not necessarily to the client that originated the conversation.**

> **Pontis preserves capability provenance across protocol boundaries.**

> **A model tool invocation may only be routed to a capability that Pontis explicitly advertised for that interaction and whose provider provenance Pontis retained.**

> **Tool names are not globally portable identities; provider, session, protocol, native name and schema form part of capability identity.**

> **Pontis performs deterministic protocol/capability mediation and routing; it does not become an agent or make behavioural decisions.**

> **Servire orchestrates Rogare operationally but does not absorb Rogare's client responsibility.**

> **Rogare may initiate and stop Trace recording for its active conversation, but Trace remains authoritative for recording lifecycle and storage.**

> **Starting or stopping Trace from Rogare must not alter the lifecycle of the underlying Rogare/Pontis conversation.**

> **Manual Rogare tracing and workflow-driven tracing must converge on the same Trace-owned recording mechanism.**

---

## 19. Resume Point

When this work is resumed, begin with the implemented Pontis/Servire architecture rather than designing Rogare in isolation.

The first concrete question should be:

**Given the final Pontis ACP session implementation, what state must Pontis retain to bind one Rogare conversation to one Pi capability-provider session and safely route advertised tool calls/results across that boundary?**

That should drive the first Rogare/Pontis capability-routing milestone.
