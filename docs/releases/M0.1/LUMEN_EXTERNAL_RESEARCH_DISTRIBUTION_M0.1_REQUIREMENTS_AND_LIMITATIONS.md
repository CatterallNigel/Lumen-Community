# Lumen External Research Distribution M0.1 — Requirements and Limitations

**Date:** 2026-08-22  
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
- establish and document what provider/model identity Vestigare currently records and whether that identity is descriptive provenance, replay-binding information, or absent from the replayable Trace;
- once established, retain sufficient provider/model execution provenance to identify the conditions under which the recorded execution occurred.

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
- distribute commands to available services;
- support simple `200`, `204`, and error response semantics.

Services with no responsibility for a command return `204`.

Services that successfully handle a command return `200`.

Nuntius is an integral Servire-owned control-plane service in M0.1. Servire starts
Nuntius automatically when Servire becomes operational, before the managed workload
stack is started. Stack Start/Stop operates on the managed workload services rather
than defining Nuntius's normal lifecycle. Servire remains able to start, stop and
restart Nuntius individually and stops Nuntius last during Servire shutdown.

Nuntius must expose its live control-plane diagnostics through its Nuntius-owned UI
surface in Servire. A reconnecting/restarted Rogare instance must be able to recover
the authoritative active Vestigare Trace state through the Nuntius control path.

### 3.7 Provider and Model Discovery

Praebere must remain the executor and authoritative state owner for model-provider operations.

For M0.1, **Ollama is the only supported and validated model-provider platform**.

Ollama must be available on its default port, `11434`. Alternative Ollama ports and alternative model-provider platforms are outside the supported M0.1 release boundary unless separately validated and explicitly brought into scope.

On startup, Praebere must query Ollama for the models actually available locally. A configured model name is an optional **preferred model**, not a requirement that the researcher's Ollama installation contain that model. If the preferred model is absent, Praebere must remain operational with no model selected; startup must not fail and another model must not be silently substituted.

M0.1 distinguishes:

- **available models** — models discovered from Ollama;
- **preferred model** — optional installation/UI preference;
- **selected model** — authoritative runtime-global execution selection;
- **selection lock** — whether active model execution prevents the selected model from being changed.

Praebere must support discovery and selection through the common `\obt` path so that Rogare and supported external clients use the same Lumen command mechanism.

For M0.1, provider and selected-model state are **runtime-global**, not session scoped. However, a model change must not alter the execution condition of an already-active session. Lumen therefore distinguishes:

- an **established session** — Pontis has assigned the authoritative `session_id`, but the session has not yet performed model interaction; merely connecting does not lock model selection;
- an **active execution session** — the session has performed its first model interaction and its model execution condition has been established.

Model selection/change is permitted only while there are no active execution sessions. Once the first model interaction occurs, the selected runtime-global model is locked until all active execution sessions have ended. Any new session established during that period uses the same locked model.

When an external client establishes a session while no model is selected/locked, it must be presented with Praebere's available models and guidance to use:

```text
\obt praebere model select <model_name>
```

The configured preferred model must **not** be silently selected for an external client.

Rogare should present Praebere's discovered models in a dropdown associated with the session controls. When selection is unlocked, an available preferred model may be shown as the initial UI choice. That choice becomes authoritative only through the normal Praebere/Nuntius selection path. While any active execution session exists, Rogare must display the authoritative model and disable model choice.

Praebere also owns the provider lifecycle. On managed stack shutdown, Servire requests Praebere lifecycle shutdown; Praebere releases/unloads the managed model resources and stops Ollama **only where the Ollama process is Praebere-managed/owned**. If Ollama was already running independently, stopping the Lumen stack must not terminate that external provider unless configuration explicitly grants Praebere that ownership.

Per-session provider/model selection remains outside the required M0.1 boundary.

### 3.8 Trace Separation

Internal `\obt` control traffic must not appear as conversational Trace turns merely because it passed through Lumen.

If a command changes a model execution condition, the resulting condition must still be represented in the execution evidence.

Examples include:

- model;
- provider;
- effective system prompt.

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
sessions do not receive independent model selections. An established session that has
not yet performed model interaction does not by itself lock selection; once any session
begins model execution, the authoritative runtime-global model is locked until all
active execution sessions have ended. New sessions established during that period use
the locked model. This must remain distinct from Moderari system-prompt policy, which is
session scoped.

### 5.7 Nuntius Is Lightweight

Nuntius is not a durable message broker.

M0.1 does not require:

- RabbitMQ;
- Kafka;
- guaranteed delivery;
- durable command queues;
- distributed consensus;
- sophisticated subscriptions.

### 5.8 Saved Prompt Management Is Minimal

M0.1 does not require:

- prompt sharing;
- organisational prompt libraries;
- prompt permissions;
- prompt version history;
- collaborative editing.

### 5.9 Mid-Session System-Prompt Policy Changes

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

The precise Vestigare representation and Repetere reconstruction rules for these transitions remain to be established when the Trace-recording changes are implemented. M0.1 must therefore treat mid-session policy changes as observable execution behaviour whose provenance must be preserved, rather than attempting to infer Replay behaviour solely from message role or position.

For research use, changing the policy mid-session should be regarded as an explicit change in experimental conditions. A researcher seeking a single controlled system-prompt condition across an experiment should avoid changing the policy during the recorded session. A researcher intentionally investigating system-prompt transitions may use this behaviour, provided the resulting Trace preserves enough evidence to reproduce the transition correctly.

Rogare does not itself supply a client system prompt. When Rogare is used as the client, the effective system prompt is therefore established through Moderari, but the same requirement remains: any policy change during a recorded session must be represented in the execution evidence.

### 5.10 Context Compaction Is Part of the Experimental Condition

Moderari's context-compaction mechanism is active in M0.1 and may materially affect model behaviour once the configured utilisation threshold is reached.

Whether compaction is desirable, neutral or detrimental is not predetermined by M0.1. Its effect may depend upon the model, context-window size, conversation, checkpoint contents and experimental objective.

Researchers should therefore treat context compaction as an observable execution condition rather than assume that a compacted session is behaviourally equivalent to the complete un-compacted conversation.

M0.1 may itself be used to investigate this behaviour, including comparison of executions before and after compaction and examination of whether checkpointing successfully preserves sufficient cognitive/contextual state.

`Pass-through` does not disable context compaction. It controls treatment of client-supplied system prompts, not Moderari's context-management behaviour.

### 5.11 Servire Operations Log Filtering

Servire's Operations Log is an operator-facing view rather than a complete copy of
every service access log.

Routine successful internal `/api/...` polling/access traffic is filtered from the
consolidated Operations Log so that model/service polling does not obscure meaningful
operator and lifecycle activity. Service-owned detailed logs remain authoritative and
are not removed by this presentation filter.

Meaningful control-plane lifecycle events, warnings, errors and unsuccessful API
responses remain visible. Nuntius command-routing and transport diagnostics remain
available through the Nuntius-owned diagnostics UI exposed within Servire.

### 5.12 Trace Model/Provider Binding Is Not Yet Established

M0.1 has not yet established whether a Vestigare Trace is operationally bound to the provider/model used for the source execution.

Development must investigate:

- whether provider identity is recorded in the Trace;
- whether model identity is recorded in the Trace;
- where and in what form that information is stored;
- whether any provider/model identity is descriptive provenance only or is used by Repetere when reconstructing a Replay;
- whether provider/model identity appears indirectly within system, assistant or other conversational messages;
- whether an otherwise unchanged Trace can currently be replayed against a different selected provider/model; and
- whether changing provider/model introduces hidden or model-specific context into the Replay.

Until this behaviour is established, M0.1 must not claim either that Replay is bound to the source provider/model or that arbitrary provider/model substitution is a supported controlled Replay condition.

### 5.13 Single-Machine Deployment

M0.1 must be installed and operated on a single host machine.

Researchers must not distribute individual Lumen services across multiple machines and expect that topology to represent a supported M0.1 research environment.

Multi-host/distributed deployment remains future development and requires separate validation of service discovery, transport, operational control, security, authorization and logging.

### 5.14 Research Distribution Is Not the Commercial Product

M0.1 does not define:

- commercial licensing;
- commercial multi-user entitlements;
- commercial operator limits;
- Aestimare assessment services;
- the final deployment/support model.

### 5.15 Runtime Protection Is Not Absolute Tamper Resistance

M0.1 runtime authorization and configuration protection are intended to prevent accidental unauthorised use, simple copying and ordinary operation outside the authorised installation model.

They do **not** make source code or locally executing software impossible for the machine owner to inspect, modify or patch.

A sufficiently determined operator with source-code and machine access may ultimately be able to bypass local checks or extract locally held material.

The security objective is therefore to make circumvention deliberate, preserve installation provenance, prevent simple redistribution from producing a functioning authorised installation, and reduce reliance on contractual enforcement. It must not be represented as absolute DRM or a hardware trust boundary.

### 5.16 Runtime Authorization Requires Periodic Illuminates.One Availability

Normal external M0.1 operation depends on periodically renewing authorization with Illuminates.One.

Temporary connectivity failure must be tolerated through the authorization lease/grace model, but the installation is not intended to operate indefinitely offline.

Offline research licensing, hardware migration, key rotation, capability manifests and the complete commercial licensing system remain future work unless separately brought into M0.1.

### 5.17 Single Active Vestigare Recording

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
- Moderari checkpoint/context-compaction behaviour remains observable in Trace evidence;
- observed mid-session system-prompt policy switching and its Trace/Replay provenance implications are documented;
- concurrent-session isolation has been demonstrated, including independent Moderari system-prompt policy state for simultaneous Rogare and Pi sessions and separation of established-session state from the new-session default;
- Vestigare enforces the M0.1 single-active-recording restriction and binds the active Trace to one authoritative Pontis `session_id`;
- Trace Start is unavailable when no active session exists;
- Vestigare presents active Pontis sessions for explicit selection when more than one eligible session exists and never guesses from recency/activity;
- M0.1 complete-session recording can begin only before the selected session's first model interaction;
- a multi-client Trace validation proves traffic from unbound sessions is excluded and the selected session identity is persisted in the Trace;
- stopping the managed stack causes Praebere to release/unload its managed model
  resources and stops Ollama only when Praebere owns the Ollama process;
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
