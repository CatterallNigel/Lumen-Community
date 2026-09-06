# Lumen External Research Distribution M0.1 — Requirements and Limitations

**Date:** 2026-09-05  
**Status:** Draft Release Definition  
**Release:** M0.1


## Document Revision History

| Date | By | Version | Description |
| --- | --- | --- | --- |
| 2026-08-23 | Nigel Catterall | 1.0 | First reviewed release |
| 2026-08-26 | Nigel Catterall | 1.1 | Added future external-client automatic provider/model discovery requirement. |
| 2026-08-26 | Nigel Catterall | 1.2 | Documented M0.1 single-active Vestigare recording restriction, ownership/session binding, and completion criterion. |
| 2026-08-27 | Nigel Catterall | 1.3 | Documented observed runtime system-prompt policy switching within an active session and the resulting Trace/Replay implications. |
| 2026-08-28 | Nigel Catterall | 1.4 | Revised Moderari to the implemented session-scoped system-prompt policy model; recorded concurrent Rogare/Pi validation, independent new-session defaults, and Servire policy-state visibility. |
| 2026-08-30 | Nigel Catterall | 1.5 | Reconciled N8+ control-plane integration: Nuntius startup/diagnostics, Trace-state reconnect recovery, Servire operational-log filtering and validation-state lifecycle, and Fiducia shutdown/PID/log-cleanup behaviour. Added failed staged-Replay recovery requirement. |
| 2026-08-30 | Nigel Catterall | 1.6 | Defined M0.1 provider/model selection as runtime-global; documented Praebere-managed stack shutdown semantics; clarified the single-active Vestigare recording limitation and required session-binding behaviour when multiple sessions/external clients exist. |
| 2026-08-30 | Nigel Catterall | 1.7 | Defined Pontis session identity as authoritative for Trace binding; required explicit active-session selection in Vestigare; prohibited Trace start with no active session; and defined M0.1 recording as beginning before the selected session's first model interaction so a Trace is not silently incomplete. |
| 2026-08-30 | Nigel Catterall | 1.8 | Refined N9 provider/model lifecycle: startup discovery from Ollama, optional preferred model, established versus active-execution sessions, external-client explicit selection, Rogare dropdown behaviour, and runtime-global model locking while execution sessions are active. |
| 2026-09-01 | Nigel Catterall | 1.9 | Added the N9.5 Pontis-owned session-management and orphan-recovery boundary, Rogare lifecycle closure, Praebere lock release/persistence/reconciliation/reset requirements, and clarified separation from the Servire authorization heartbeat. |
| 2026-09-04 | Nigel Catterall | 2.0 | Added N9.6.2 deferred-release reconciliation and N9.6.3 external-Ollama, demand-residency and atomic runtime-model reservation requirements. Removed Praebere ownership of the Ollama process lifecycle. |
| 2026-09-05 | Nigel Catterall | 2.1 | Clarified Praebere authority for model selection, reservation and residency; retained Pontis authority for session identity/lifecycle; defined available, reserved and locked selection states; and clarified that Pontis `not active` means not active in model execution, not that the session is closed or the model is available. |
| 2026-09-06 | Nigel Catterall | 2.2 | Defined the M0.1 cached model-discovery and explicit-refresh boundary, including stale-cache visibility and failed-load reservation cancellation. |
| 2026-09-06 | Nigel Catterall | 2.3 | Reconciled successful Vestigare provenance validation; required authoritative model metadata in `trace_recordings`; defined Repetere/Fiducia exact recorded-model enforcement under the runtime-global M0.1 model boundary; and documented Rogare's manual session-reconnect limitation. |

## 1. Purpose

M0.1 is the first defined milestone for the Lumen External Research Distribution.

Its purpose is to provide a coherent research baseline for observing, reproducing and repeatedly executing model behaviour under explicit experimental conditions.

M0.1 is not intended to represent the complete Lumen product or the eventual commercial distribution.

Aestimare is not part of M0.1.

## 2. Included Lumen Services

The intended M0.1 stack comprises:

- Rogare;
- Pontis;
- Vestigare;
- Repetere;
- Fiducia;
- Moderari;
- Praebere;
- Servire;
- Nuntius;
- MongoDB as supporting persistence.

The distribution is intended to operate as a Dockerised Lumen stack over a private Lumen Docker network.

M0.1 is a **single-machine distribution**. All Lumen services, supporting services and the M0.1 Docker environment must be installed and operated on the same host machine.

The Dockerised distribution exposes only the host entry points required for operation and research access:

- **Pontis:** host port `11435` — the Lumen entry point for supported external clients;
- **Servire:** host port `11439` — the operational/control UI entry point;
- **MongoDB:** host port `27018` — the research/data access entry point to Lumen's persisted experimental and Trace data.

MongoDB continues to use its normal container-side port `27017`; the M0.1 Docker configuration maps host port `27018` to container port `27017`.

Using `27018` as the host-side MongoDB port reduces the likelihood of conflict with an existing MongoDB installation already using the standard host port `27017`. It also allows a researcher to connect directly to Lumen's MongoDB data with their own preferred database tools or locally developed research/visualisation tooling.

All other Lumen service communication is intended to remain within the private Lumen Docker network unless explicitly documented otherwise.

Although Lumen's service architecture may permit components to communicate across network boundaries, distributed multi-host deployment is not supported or validated in M0.1.

## 3. Functional Requirements

### 3.1 Model Execution and Trace

M0.1 must:

- execute model interactions through the Lumen stack;
- record the effective model execution through Vestigare;
- preserve the actual context supplied to the model;
- preserve the effective system prompt;
- retain the incoming client system prompt as superseded provenance when Moderari
  replaces it, exclude that superseded prompt from Replay input, and identify exactly
  one authoritative effective system prompt as replayable execution context;
- store the authoritative provider and exact model identity used for execution in the
  corresponding Vestigare `trace_recordings` document; and
- retain sufficient provider/model execution provenance to identify and enforce the
  model condition under which the recorded execution occurred.

### 3.2 Experiment and Repetition

M0.1 must:

- represent an Experiment explicitly;
- associate an Experiment with a source Trace;
- associate repeated replay executions with that Experiment;
- allow Fiducia to coordinate repeated executions;
- allow Repetere to perform each reproduction;
- retain each resulting execution as a normal Vestigare Trace;
- display Experiment runs as related evidence.

Each run must expose at least:

- `MATCHED`;
- `DIVERGED`; or
- `FAILED / INCOMPLETE`.

Where existing evidence permits, the first divergence point should be exposed.

A failed Replay run must remain preserved as experimental evidence while the staged
Experiment remains recoverable. Retrying must create a fresh isolated Replay
session/run and must not overwrite or silently discard the failed-run evidence.

Repetere reports divergence but does not assess its significance.

At actual replay start, Repetere must read the authoritative provider/model identity
from the source `trace_recordings` document and apply the normal Pontis/Praebere
session-selection lifecycle:

- when no global model is selected, the recorded model must exist in Praebere's cached
  catalogue and must be selected/reserved for the Replay session before execution;
- when the global model exactly matches the recorded model, Repetere must reserve that
  model for the Replay session and continue normally;
- when a different global model is selected, reserved or locked, Repetere must log and
  display a controlled model-conflict error and must not start the Replay;
- when authoritative model metadata is absent or the recorded model is unavailable,
  Repetere must fail explicitly before model execution; and
- Repetere must never load Ollama models directly or silently substitute the active
  model. First-execution activation remains Praebere's responsibility.

Fiducia must apply the same compatibility check at scheduled execution time. If the
active global model differs from the recorded model, or the recorded model is missing
or unavailable, Fiducia must record a failed scheduled-run result and must not change
or interrupt the active global model.

### 3.3 System-Prompt Policy

Moderari must support:

- `Pass-through`;
- `Moderari Default`;
- `Custom`.

`Moderari Default` remains the normal/default behaviour.

Pass-through must genuinely leave the client-supplied system context unchanged.

Custom mode must allow a researcher-defined prompt to replace the incoming client system prompt.

### 3.4 Context Compaction

Moderari manages context utilisation during an active session.

When context utilisation exceeds approximately **63% of the available model context window**, Moderari creates a checkpoint representing a compacted form of the model's current cognitive/contextual state.

The purpose of compaction is to allow a session to continue beyond the point at which retaining and repeatedly supplying the complete accumulated conversation would consume too much of the model's finite context window. Rather than simply discarding older context, Moderari attempts to preserve the significant cognitive/contextual state required for continuity in a more compact representation.

As the session continues, Moderari uses checkpointing to preserve significant conversational state while reducing the amount of historical context that must continue to be supplied directly to the model. The **two most recent checkpoints** are returned as part of the continuing context.

Context compaction is independent of Moderari's system-prompt policy.

In particular:

- `Pass-through` means that Moderari does not alter the client-supplied system prompt; it does **not** disable Moderari context management;
- `Moderari Default` and `Custom` determine the system-prompt condition;
- context compaction determines how an extended conversation is represented after the configured utilisation threshold is reached.

A sufficiently long session may therefore cease to present the complete original conversational history verbatim and instead present the context constructed through Moderari's checkpoint/compaction mechanism.

Vestigare must record the effective execution context actually presented to the model so that checkpointing and compaction remain observable as part of the Trace evidence.

### 3.5 Saved System Prompts

M0.1 must allow custom system prompts to be:

- saved under a name;
- listed;
- selected for reuse;
- edited;
- deleted.

The actual prompt injected into the model remains part of Trace evidence.

### 3.6 `\obt` Control Commands

M0.1 establishes `\obt` as the common Lumen control-command language.

Pontis must forward `\obt` control traffic to Nuntius rather than send it through normal ask/answer processing.

Nuntius must:

- obtain the active service catalogue from Servire;
- accept an empty catalogue as healthy when Servire is reachable;
- maintain the available-service catalogue;
- resolve the configured command owner and route directly to that service rather than
  broadcasting or using legacy consume-or-forward traversal;
- support simple `200`, `204`, and error response semantics.

Services that successfully handle a command return `200`.

A routed owner may return `204` only where the defined operation legitimately has no
response body. Silence, a timeout or the absence of an authoritative outcome must never
be interpreted as success.

Nuntius is an integral Servire-owned control-plane service in M0.1. Servire starts
Nuntius automatically when Servire becomes operational, before the managed workload
stack is started. Stack Start/Stop operates on the managed workload services rather
than defining Nuntius's normal lifecycle. Servire remains able to start, stop and
restart Nuntius individually and stops Nuntius last during Servire shutdown.

Nuntius must expose its live control-plane diagnostics through its Nuntius-owned UI
surface in Servire. A reconnecting/restarted Rogare instance must be able to recover
the authoritative active Vestigare Trace state through the Nuntius control path.

### 3.7 Provider and Model Discovery

Praebere must remain the executor and authoritative state owner for model discovery,
selection intent, demand loading and Lumen-owned model residency.

For M0.1, **Ollama is the only supported and validated model-provider platform**.

Ollama is external infrastructure and must already be available at the configured
endpoint. The M0.1 default is port `11434`. Praebere must not start, stop or otherwise
assume process ownership of Ollama. If the configured endpoint is unavailable during
Praebere lifecycle startup, startup must fail explicitly and Servire must roll back the
services started by that stack-start attempt. Alternative model-provider platforms are
outside the supported M0.1 release boundary unless separately validated and explicitly
brought into scope.

On startup, Praebere must query Ollama for the models actually available locally and
persist the resulting model catalogue as authoritative cached discovery state. Ordinary
model-list queries, model-selection validation, status reads and Praebere UI polling must
use that cache and must not repeatedly query Ollama. The Praebere UI **Refresh Models**
action is the explicit mechanism for replacing the cache with current Ollama discovery.
If an explicit refresh fails, Praebere must retain the last successfully discovered
catalogue and report the refresh failure rather than silently replacing the catalogue
with an empty or partial result.

A configured model name is an optional **preferred model**, not a requirement that the
researcher's Ollama installation contain that model. If the preferred model is absent,
Praebere must remain operational with no model selected; startup must not fail and another
model must not be silently substituted.

M0.1 distinguishes:

- **available models** — models discovered from Ollama;
- **preferred model** — optional installation/UI preference;
- **selected model** — authoritative runtime-global execution selection;
- **model reservation** — the runtime-global model atomically reserved by Praebere
  for one or more open Pontis sessions;
- **active execution session** — a session whose first model ask has activated
  execution through Praebere;
- **Praebere-owned residency** — a resident model that Praebere itself requested
  Ollama to load.

Praebere must support discovery and selection through the common `\obt` path so that Rogare and supported external clients use the same Lumen command mechanism.

For M0.1, provider and selected-model state are **runtime-global**, not session scoped.
Model selection and model loading are separate operations. Selecting a model records
intent and atomically reserves that model in Praebere for the requesting Pontis
session; it must not load the model.
Lumen therefore distinguishes:

- an **established session** — Pontis has assigned the authoritative `session_id`; the
  session may have no model reservation or may reserve the runtime-global selected model;
- an **active execution session** — the session has performed its first model interaction and its model execution condition has been established.

Pontis UI/API text `not active` means **not active in model execution**. It does not
mean that the Pontis session is closed, and it does not by itself mean that model
selection is available. An established, non-executing session may still reserve the
runtime-global model in Praebere.

Praebere exposes the authoritative model-selection state:

- **available** — no open session currently reserves the selected model;
- **reserved** — one or more open sessions reserve the selected model, but none is
  currently active in model execution;
- **locked** — one or more sessions are active in model execution; and
- **reconciliation pending** — Praebere cannot yet safely determine the authoritative
  reservation/execution state.

The first successful selection reserves the runtime-global model for that session.
Other sessions may reserve the same model. A request for a different model must be
rejected atomically by Praebere while any reservation exists. If two
clients concurrently request different models, exactly one model may win; both clients
must display Praebere's authoritative confirmed model rather than an optimistic local
choice. Pontis records the confirmed model for routing and presentation but does not
independently create or enforce the model reservation.

Praebere loads the selected model only on the first ask, and only if Ollama does not
already report it resident. Praebere records residency ownership only when it issued
that load. Multiple execution sessions using the reserved model share the residency.
If Ollama cannot load the selected model, including because it was removed after the
last successful discovery, Praebere must return a controlled model-activation error,
must not register the execution session as active, and must cancel and persist the
requesting session's reservation. The client must be told that both activation failed
and its reservation was cancelled. Reservations held by other open sessions must remain
unchanged; the resulting selection state is `available` when no reservation remains and
`reserved` when another reservation remains.
When the final active execution session ends, Praebere unloads the model only if it
owns that residency. A model that was already resident before Lumen used it remains
loaded. When the final reserving session ends, Pontis reports the session-lifecycle
change and Praebere releases that session's reservation; the last selected model may
remain recorded as history but is no longer reserved.

When an external client establishes a session while no model is reserved, it must be
presented with Praebere's available models and guidance to use:

```text
\obt praebere model select <model_name>
```

The configured preferred model must **not** be silently selected for an external client.

Rogare must expose Praebere's authoritative model discovery and selection through the
same `\obt praebere` control path used by external clients. A model becomes authoritative
only after Praebere accepts the session's normal selection/reservation request. While a
reservation exists, Rogare must display the authoritative reserved model and surface a
controlled conflict if that session requests a different model.

On managed stack shutdown, Praebere releases only model residency that it caused.
Lumen must never stop Ollama. Failure to unload Lumen-owned residency must be reported
as degraded partial completion and must not be concealed by terminating the provider.

Per-session provider/model selection remains outside the required M0.1 boundary.

### 3.7.1 Session Management and Model-Lock Release

Pontis is authoritative for session identity and lifecycle. M0.1 must provide:

- a Pontis-owned Servire UI showing established and active-execution sessions,
  origin, request/in-flight state, timestamps and Praebere registration/release
  state;
- graceful End Session and confirmed, authorised Force Close / Clear Orphan
  operations;
- client-scoped `\obt pontis session status`, `session end` and `session new`
  commands;
- restricted administrative `\obt pontis sessions`, named `session end` and
  `session force-close` commands;
- an End Session action in Rogare;
- termination of the current Rogare session during New Session, normal stop and
  restart; and
- idempotent, auditable release of the ended session's Praebere execution
  registration through Nuntius.

If Praebere is unavailable, local Pontis closure is partial success: the session must
be shown as `release pending`, and Force Close feedback must state both the successful
local closure and deferred provider release. On Praebere restart, reconciliation must
run automatically, consume pending releases, update Praebere's active set, acknowledge
Pontis and reach `complete` without requiring another client request.

Ordinary external clients may inspect, end or replace only their own Pontis
session. Administrative cross-session operations must be authorised. Idle time
alone must not classify or close a session as orphaned. M0.1 does not require
Pontis to POST a heartbeat to an external-client callback address.

Praebere must release its execution lock and any Praebere-owned residency only after
the final **active execution session** has ended. Praebere must retain the runtime-model
reservation until the final Pontis session reserving that model has ended. Praebere must
persist selected provider/model state, residency ownership and active execution
registrations, restore them fail-safe after restart and reconcile against Pontis
authority. A Pontis session closed while Praebere is unavailable must remain visibly
`release pending`; Praebere restart must reconcile it, remove it from the active set,
acknowledge release to Pontis and reach `complete`. Safe execution reset is
permitted only when Pontis confirms no active execution sessions remain; forced
reset, if provided, is restricted and auditable emergency recovery.

The mandatory Servire distribution-authorization heartbeat remains a distinct
licensing/security requirement. It neither proves client-session liveness nor
replaces Pontis session termination.

### 3.8 Trace Separation

Internal `\obt` control traffic must not appear as conversational Trace turns merely because it passed through Lumen.

If a command changes a model execution condition, the resulting condition must still be represented in the execution evidence.

Examples include:

- model;
- provider;
- effective system prompt.

Vestigare recording metadata must identify the authoritative provider and exact model
used. Individual request/response bodies may also contain the model identity, but
Repetere and Fiducia must not be required to decode conversational transport bodies to
determine the source execution model.

### 3.9 Concurrent Session Isolation

M0.1 must be validated with multiple simultaneous Lumen sessions.

Transactions within concurrent sessions must remain bounded to their originating session. Session-specific conversational context, responses, Trace evidence and other session-specific state must not leak between sessions.

For M0.1, conversational state and Moderari system-prompt policy are **session scoped**.

Live concurrent-session validation with Rogare and Pi demonstrated that one active session can remain under `Moderari Default` while another active session is independently changed to `Pass-through`. The change applies only to the selected/originating session and does not alter the policy of another active session.

Moderari distinguishes between the **default policy for new sessions** and the **active policy of each established session**. Changing an established session's policy does not silently change the new-session default and does not alter another established session.

Provider/model concurrency remains subject to separate M0.1 validation and limitations; the demonstrated session-scoped system-prompt behaviour must not be generalized into an unsupported provider/model claim.

### 3.10 Client and Tool Responsibility

M0.1 supports two distinct client/tool responsibility paths.

#### External Clients

When an external client is used, that client is responsible for its own tool environment.

This includes responsibility for:

- declaring or exposing the tools available to the model;
- providing any tool definitions required by the client/model interaction;
- executing tool calls;
- returning tool results into the continuing client/model interaction.

Lumen does not prescribe or manage an external client's tool environment in M0.1.

From Lumen's perspective, client-managed tools form part of the external client's execution environment.

M0.1 therefore makes no claim of compatibility with, or responsibility for, arbitrary external-client tool implementations.

#### Rogare

Rogare has no native tool environment.

Where tool use is required, Rogare relies upon an external tool provider.

For M0.1:

- Pi is the only supported and validated tool provider for Rogare;
- Pi must be installed in its expected/default installation location;
- Rogare uses the existing Lumen integration to access Pi;
- alternative tool providers are not supported through Rogare in M0.1.

Broader Rogare tool-provider discovery, configurable provider locations, multiple simultaneous providers and alternative tool-provider integrations remain outside the M0.1 release boundary unless separately validated and brought into scope.

### 3.11 Runtime Authorization and Code Protection

M0.1 must implement the runtime authorization boundary defined by **M0.1 Runtime Authorization and Code Protection**.

For the external research distribution:

- possession of the distributed source code must not, by itself, authorize operation of a Lumen installation;
- each installation must have a Service Group UUID and a distinct cryptographic installation identity;
- the Service Group UUID is an identifier and must not be treated as an authentication secret;
- Servire must establish and maintain runtime authorization with Illuminates.One over TLS/HTTPS;
- Servire authorization requests must be attributable to the installation cryptographic identity;
- Illuminates.One authorization responses must be cryptographically signed and independently verifiable by Servire;
- authorization exchanges must be freshness-protected, including a nonce or equivalent established mechanism;
- runtime authorization must operate as a time-limited lease with renewal and a bounded grace state;
- an installation must not continue authorised operation indefinitely after authorization expires;
- deliberate heartbeat suppression must not provide a route to indefinite operation;
- Servire must be the supported operational authority for starting, supervising, stopping and restarting protected Lumen services;
- protected services must be able to determine that startup was authorised by Servire;
- direct command-line invocation of an individual protected service is not a supported external M0.1 runtime path;
- protected service configuration must be encrypted at rest where it contains protected operational information;
- successful runtime authorization must contribute to making protected configuration usable;
- the distributed installation must not contain a permanent standalone secret that is sufficient by itself to unlock all protected runtime configuration;
- decrypted protected configuration should remain in memory where practical and must not be routinely persisted as plaintext;
- copying the distributed source, containers and configuration to another machine must not automatically create another authorised Lumen installation.

Individual services must not independently implement the Illuminates.One licensing/authorization relationship. That trust boundary remains concentrated in Servire.

The detailed security architecture, key ownership, authorization exchange, lease model and limitations are defined in the dedicated security document rather than duplicated here.

## 4. Research Requirements

M0.1 must allow experimental conditions to be explicit enough that repeated executions can be meaningfully compared.

The release must preserve the principle:

> **Experimental conditions should be explicit, reproducible and observable. Lumen should record what happened without unnecessarily altering the conditions being investigated.**

M0.1 provides evidence and objective replay-divergence information.

It does not provide behavioural assessment.

## 5. Known Limitations

### 5.1 No Aestimare

Aestimare is explicitly excluded.

M0.1 does not determine whether a divergence is significant, desirable, undesirable, trustworthy or untrustworthy.

### 5.2 No Human Response Rating

Rogare does not yet collect independent 1–10 correctness and quality ratings from researchers.

This is future work intended eventually to provide human assessment evidence for comparison with Aestimare.

### 5.3 Single Lumen Tool Provider Assumption

The Lumen-managed tool path may remain limited to one attached tool provider in M0.1.

For Rogare, the supported and validated provider is Pi in its expected/default installation location.

This limitation does not apply to tool environments owned and executed by an external client. Those remain the responsibility of that client and are outside Lumen's M0.1 tool-provider compatibility claim.

The architecture does not define the Lumen-managed tool path as permanently single-provider.

Multiple providers, provider discovery, namespaces and a unified tool catalogue remain future development.

### 5.4 No Resource-Bound Behaviour Experimentation Feature

M0.1 does not provide dedicated facilities for controlled experiments varying computational resource envelopes.

This remains a research question rather than an M0.1 feature.

### 5.5 No General Multi-Operator Servire Access

M0.1's successful multi-session validation does not imply that the Research Distribution is intended for unrestricted simultaneous use by multiple researchers.

Before general external research release, Servire is expected to enforce a single active operator session per installation.

That enforcement is not an M0.1 requirement.

### 5.6 Concurrent Execution Conditions

Moderari system-prompt policy is no longer a shared global execution condition.

M0.1 now supports independently established **session-scoped Moderari system-prompt policy**. Concurrent live validation demonstrated a Rogare session remaining under `Moderari Default` while a Pi/external-client session was changed to `Pass-through`. Policy status queries from each session reported its own state, and Servire displayed the selected session's policy independently from the default used for new sessions.

The following distinctions therefore apply:

- **new-session default** — the policy inherited when a new Moderari session is established;
- **session active policy** — the policy currently applied to model requests for that established session;
- changing one session's active policy does not change another session;
- changing one session's active policy does not change the new-session default.

Provider/model selection is deliberately **runtime-global for M0.1**. Concurrent
sessions do not receive independent model selections. Praebere atomically reserves the
first successfully selected model for the selecting Pontis session before execution begins.
Other sessions may join that same reservation but cannot reserve a different model
until every reserving session has ended. First ask activates execution and demand-loads
the reserved model; it does not decide which client wins model selection. This must
remain distinct from Moderari system-prompt policy, which is session scoped.

### 5.7 Model Discovery Is Cached

Praebere's model catalogue is a snapshot from startup or the most recent successful
explicit **Refresh Models** action. It is not a continuously synchronised view of
Ollama. Models added to or removed from Ollama outside Lumen are therefore not reflected
in ordinary `\obt praebere models` responses or the Praebere UI until the researcher
uses **Refresh Models** or restarts the stack.

A model appearing as `available` in Praebere means that it existed at the last successful
discovery; it does not guarantee that the model still exists or can currently be loaded.
If Ollama cannot load a cached model on first ask, the ask fails with a controlled
activation error and the requesting session's reservation is cancelled. The researcher
must correct the Ollama condition or refresh the catalogue before selecting again.

If live refresh cannot reach Ollama or otherwise fails, Praebere deliberately retains
the previous catalogue and displays the refresh failure. This fail-safe behaviour avoids
silently discarding the last known discovery state, but the retained catalogue may be
stale until a later refresh succeeds.

### 5.8 Nuntius Is Lightweight

Nuntius is not a durable message broker.

M0.1 does not require:

- RabbitMQ;
- Kafka;
- guaranteed delivery;
- durable command queues;
- distributed consensus;
- sophisticated subscriptions.

### 5.9 Saved Prompt Management Is Minimal

M0.1 does not require:

- prompt sharing;
- organisational prompt libraries;
- prompt permissions;
- prompt version history;
- collaborative editing.

### 5.10 Mid-Session System-Prompt Policy Changes

Moderari system-prompt policy is evaluated at model-request time rather than being fixed for the lifetime of an external-client session.

Current testing with Pi has demonstrated that an active session can transition between `Moderari Default` and `Pass-through` without restarting the client session. Concurrent Rogare/Pi testing additionally demonstrated that this transition is session scoped: changing the Pi session to `Pass-through` left the already-active Rogare session on `Moderari Default`, while the default for newly created sessions also remained `Moderari Default`. A subsequent request is processed using the newly selected policy while the existing conversational history remains available. For example, a session may begin under `Moderari Default`, change to `Pass-through`, and later return to `Moderari Default`, with the model retaining conversational continuity across those transitions.

This is valid and potentially useful research behaviour, but it creates an important Trace and Replay consideration. A single recorded session may contain model executions produced under different system-prompt policies and therefore under different effective system prompts. The policy transition is part of the experimental condition and must not be treated as though one system prompt governed the entire session.

In particular, an external client may continue to carry its own system message in the conversation while Moderari later applies `Moderari Default` or `Custom`. Depending on how Vestigare records the incoming client context and the effective model context, a Trace may therefore contain evidence of both the client-supplied system prompt and a Moderari-applied system prompt.

This has implications for Repetere. Replay must not assume that every recorded client system message should always be retained, nor that every client system message should always be suppressed when a Moderari prompt is present. If Vestigare records the external client's system prompt and also records Moderari's effective replacement, Repetere requires sufficient provenance to distinguish:

- the system context supplied by the external client;
- the system-prompt policy active for each model execution;
- the effective system prompt actually supplied to the model; and
- any policy transition occurring during the recorded session.

Without that distinction, Replay could incorrectly send both the external-client system prompt and the Moderari-generated prompt, or incorrectly remove a client system prompt that was intentionally active during a `Pass-through` portion of the session.

Vestigare now records authoritative system-prompt provenance for each applicable
exchange. The incoming client prompt is retained with `disposition: superseded` and is
not Replay input when Moderari replaces it. The Lumen/Moderari effective system prompt
is retained separately and identified as the single replayable system prompt. Replay
must continue to use this explicit provenance rather than infer behaviour solely from
message role or position.

For research use, changing the policy mid-session should be regarded as an explicit change in experimental conditions. A researcher seeking a single controlled system-prompt condition across an experiment should avoid changing the policy during the recorded session. A researcher intentionally investigating system-prompt transitions may use this behaviour, provided the resulting Trace preserves enough evidence to reproduce the transition correctly.

Rogare does not itself supply a client system prompt. When Rogare is used as the client, the effective system prompt is therefore established through Moderari, but the same requirement remains: any policy change during a recorded session must be represented in the execution evidence.

### 5.11 Context Compaction Is Part of the Experimental Condition

Moderari's context-compaction mechanism is active in M0.1 and may materially affect model behaviour once the configured utilisation threshold is reached.

Whether compaction is desirable, neutral or detrimental is not predetermined by M0.1. Its effect may depend upon the model, context-window size, conversation, checkpoint contents and experimental objective.

Researchers should therefore treat context compaction as an observable execution condition rather than assume that a compacted session is behaviourally equivalent to the complete un-compacted conversation.

M0.1 may itself be used to investigate this behaviour, including comparison of executions before and after compaction and examination of whether checkpointing successfully preserves sufficient cognitive/contextual state.

`Pass-through` does not disable context compaction. It controls treatment of client-supplied system prompts, not Moderari's context-management behaviour.

### 5.12 Servire Operations Log Filtering

Servire's Operations Log is an operator-facing view rather than a complete copy of
every service access log.

Routine successful internal `/api/...` polling/access traffic is filtered from the
consolidated Operations Log so that model/service polling does not obscure meaningful
operator and lifecycle activity. Service-owned detailed logs remain authoritative and
are not removed by this presentation filter.

Meaningful control-plane lifecycle events, warnings, errors and unsuccessful API
responses remain visible. Nuntius command-routing and transport diagnostics remain
available through the Nuntius-owned diagnostics UI exposed within Servire.

### 5.13 Replay Uses the Runtime-Global Model Boundary

M0.1 does not provide a private or per-session model for Repetere. A Replay must use the
model recorded for its source Trace, but that model must also be compatible with the
current runtime-global Praebere selection.

The exact model appears in recorded request and response transport bodies. M0.1 must
add the authoritative provider and exact model identity directly to the associated
`trace_recordings` document so Replay does not infer it from message content.

At Replay start:

- if no model is selected, Repetere may select/reserve the recorded model through the
  ordinary Pontis/Praebere lifecycle;
- if the selected model matches exactly, Replay may share it as another normal session;
- if a different model is selected, reserved or locked, Replay must stop with a clear
  conflict rather than substitute that model or disturb active sessions.

Consequently, an M0.1 Replay may be delayed until sessions reserving a different model
have ended. Running a Replay concurrently on a different, Replay-private model remains
deferred until Lumen implements general per-session model selection and multi-model
residency management.

### 5.14 Single-Machine Deployment

M0.1 must be installed and operated on a single host machine.

Researchers must not distribute individual Lumen services across multiple machines and expect that topology to represent a supported M0.1 research environment.

Multi-host/distributed deployment remains future development and requires separate validation of service discovery, transport, operational control, security, authorization and logging.

### 5.15 Research Distribution Is Not the Commercial Product

M0.1 does not define:

- commercial licensing;
- commercial multi-user entitlements;
- commercial operator limits;
- Aestimare assessment services;
- the final deployment/support model.

### 5.16 Runtime Protection Is Not Absolute Tamper Resistance

M0.1 runtime authorization and configuration protection are intended to prevent accidental unauthorised use, simple copying and ordinary operation outside the authorised installation model.

They do **not** make source code or locally executing software impossible for the machine owner to inspect, modify or patch.

A sufficiently determined operator with source-code and machine access may ultimately be able to bypass local checks or extract locally held material.

The security objective is therefore to make circumvention deliberate, preserve installation provenance, prevent simple redistribution from producing a functioning authorised installation, and reduce reliance on contractual enforcement. It must not be represented as absolute DRM or a hardware trust boundary.

### 5.17 Runtime Authorization Requires Periodic Illuminates.One Availability

Normal external M0.1 operation depends on periodically renewing authorization with Illuminates.One.

Temporary connectivity failure must be tolerated through the authorization lease/grace model, but the installation is not intended to operate indefinitely offline.

Offline research licensing, hardware migration, key rotation, capability manifests and the complete commercial licensing system remain future work unless separately brought into M0.1.

### 5.18 Single Active Vestigare Recording

M0.1 supports only **one active Vestigare recording at a time** across the Lumen
installation, regardless of how many Rogare or external-client sessions are active.

Pontis is authoritative for Lumen session identity. Every supported session, including
an otherwise unnamed external-client session, is assigned a Pontis `session_id`, and
session-correlated traffic traversing the model path carries that identity and its
associated session metadata.

Vestigare must never infer recording ownership from the "last-opened", "most recent"
or "currently busiest" session. An active Trace is bound explicitly to one Pontis
`session_id`:

```text
trace_id -> Pontis session_id
```

Traffic for other concurrent sessions continues normally but must not be incorporated
into the active Trace.

#### Trace Start and Session Selection

A Trace cannot be started when there are **no active Lumen sessions**. The recording
control must be disabled or the request rejected clearly because there is no session
to which the Trace can be bound.

When exactly one eligible active session exists, that session may be selected
automatically, but the UI must display the actual Pontis session identity being
recorded.

When more than one eligible active session exists, Vestigare must present the active
Pontis sessions and require the researcher to select which session is to be recorded.
M0.1 must not silently choose a session.

Where Trace Start originates from a session-aware control path, the originating Pontis
`session_id` may be supplied as the proposed selection, but the resulting recording
must still persist that explicit session binding.

Once a Trace is active, attempts to start another recording must be prevented.

#### Recording Start Boundary

For M0.1, a research Trace intended to represent a complete session must begin
**before the selected session's first model interaction**.

Starting Vestigare after a session has already exchanged model traffic would otherwise
miss the earlier execution sequence. A later request may carry conversational context
containing earlier messages, but that is not equivalent to having observed and
recorded the earlier requests, responses, tool activity, execution conditions and
other provenance as they occurred.

M0.1 must therefore not silently represent a mid-session recording as a complete
session Trace. Trace Start is permitted only for an eligible active session that has
not yet performed its first model interaction.

This supports the normal sequence: the client connects, Pontis establishes and names
the session, the researcher starts Vestigare for that session, and only then submits
the first research prompt.

Explicit partial/mid-session Trace support, historical backfilling, and multiple
simultaneous Vestigare recordings are outside the required M0.1 boundary.

**M0.1 acceptance:** create at least two simultaneous eligible sessions, confirm that
Vestigare presents both Pontis session identities for selection, select one session,
start recording before its first model interaction, send traffic through both sessions,
and confirm that only the bound session appears in the Trace. Also verify that Trace
Start is unavailable with no active session and is rejected for a session that has
already begun model interaction.

### 5.19 Rogare Session Reconnect Is Manual

Rogare does not automatically discover or reconnect to an existing Pontis session in
M0.1. After Rogare restart or disconnection, the researcher must obtain the active
session ID from Pontis, enter it in Rogare's Session field and start using that session.

The authoritative Pontis session and its model/conversational context continue, but
Rogare does not restore its prior local conversation display. Earlier messages may
therefore be absent from the Rogare UI even though the continuing model request retains
the established Lumen session context.

Automatic session discovery and restoration of Rogare conversation presentation remain
future improvements. This limitation does not indicate loss of Pontis session identity
or model context.

## 6. Pre-General-Research-Release Requirement

Before the Research Distribution is made generally available to external researchers, Servire should enforce a single active operator session per installation.

The mechanism should:

- permit legitimate remote browser operation;
- decline a second simultaneous operator cleanly;
- explain that an active session already exists;
- provide an explicit Exit/Release Session action;
- use a renewable lease/timeout so abandoned sessions do not permanently lock the installation.

This is a distribution/access policy and must not be confused with Lumen's underlying ability to isolate concurrent sessions.

## 7. M0.1 Completion Criteria

M0.1 can be considered complete when:

- the Experiment structure works end-to-end;
- Fiducia coordinates repeated Experiment runs;
- Repetere divergence is visible;
- Moderari system-prompt policy is configurable;
- saved custom prompts can be reused;
- external-client tool declaration and execution remain the responsibility of the external client;
- Rogare tool use is validated against the supported Pi provider in its expected/default installation location;
- Nuntius provides the common `\obt` distribution path;
- Praebere discovery/selection uses the agreed control path;
- `\obt` traffic remains outside conversational Trace;
- effective execution context remains observable;
- provider/model Trace provenance and any Replay binding behaviour have been established and documented before a provider/model Replay guarantee is made;
- every new Vestigare `trace_recordings` document records the authoritative provider
  and exact model used for execution;
- Repetere verifies and reserves the recorded model at actual Replay start, proceeds
  only when the runtime-global selection matches, and fails clearly without model
  execution when it conflicts or is unavailable;
- Fiducia records a failed scheduled-run result instead of changing the global model
  when the scheduled Trace model conflicts with the active selection;
- Moderari checkpoint/context-compaction behaviour remains observable in Trace evidence;
- observed mid-session system-prompt policy switching and its Trace/Replay provenance implications are documented;
- concurrent-session isolation has been demonstrated, including independent Moderari system-prompt policy state for simultaneous Rogare and Pi sessions and separation of established-session state from the new-session default;
- Vestigare enforces the M0.1 single-active-recording restriction and binds the active Trace to one authoritative Pontis `session_id`;
- Trace Start is unavailable when no active session exists;
- Vestigare presents active Pontis sessions for explicit selection when more than one eligible session exists and never guesses from recency/activity;
- M0.1 complete-session recording can begin only before the selected session's first model interaction;
- a multi-client Trace validation proves traffic from unbound sessions is excluded and the selected session identity is persisted in the Trace;
- a Praebere restart validation proves deferred Pontis releases reconcile
  automatically and reconciliation reaches `complete` without a further client ask;
- Force Close reports local closure plus deferred Praebere release as partial success;
- Praebere startup fails explicitly when the configured Ollama endpoint is unavailable,
  causing Servire to roll back that stack-start attempt;
- selecting a model does not load it; the first ask demand-loads it when absent;
- Praebere atomically enforces one runtime-global model reservation while allowing
  concurrent sessions to share the same reserved model;
- clients display the Praebere-confirmed authoritative model after selection success or
  conflict, including the current reserved model in human-readable conflict feedback;
- stopping the managed stack causes Praebere to unload only model residency that it
  caused and never stops externally operated Ollama;
- the complete stack passes integration/regression validation as a single-host installation;
- installation identity and signed runtime authorization work end-to-end;
- authorization lease renewal and expiry behaviour have been demonstrated;
- protected services reject unsupported/unauthorised startup paths as defined by the security design;
- protected configuration cannot be used through the normal external runtime path without valid authorization;
- copying an installation does not, by itself, create another authorised installation;
- no Illuminates.One private signing key, permanent master decryption key or equivalent secret is present in the distributed release;
- known limitations are documented.

## 8. Release Boundary

M0.1 should remain deliberately bounded.

Features should not be added merely because they are desirable for the eventual Lumen ecosystem.

The milestone is complete when it provides a stable research baseline for:

> **Observe → Reproduce → Repeat**

with explicit experimental conditions and sufficient provenance to support later assessment.

Aestimare and broader Reasoning Assurance capabilities build upon that evidence; they do not need to be pulled into M0.1.
