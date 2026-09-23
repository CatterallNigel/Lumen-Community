# Lumen External Research Distribution M0.1 --- Requirements and Limitations

**Date:** 2026-09-14\
**Status:** FINAL --- M0.1 release baseline validated 2026-09-23
**Release:** M0.1

## Document Revision History

  --------------------------------------------------------------------------------------------------
  Date         By          Version   Description
  ------------ ----------- --------- ---------------------------------------------------------------
  2026-08-23   Nigel       1.0       First reviewed release
               Catterall             

  2026-08-26   Nigel       1.1       Added future external-client automatic provider/model discovery
               Catterall             requirement.

  2026-08-26   Nigel       1.2       Documented M0.1 single-active Vestigare recording restriction,
               Catterall             ownership/session binding, and completion criterion.

  2026-08-27   Nigel       1.3       Documented observed runtime system-prompt policy switching
               Catterall             within an active session and the resulting Trace/Replay
                                     implications.

  2026-08-28   Nigel       1.4       Revised Moderari to the implemented session-scoped
               Catterall             system-prompt policy model; recorded concurrent Rogare/Pi
                                     validation, independent new-session defaults, and Servire
                                     policy-state visibility.

  2026-08-30   Nigel       1.5       Reconciled N8+ control-plane integration: Nuntius
               Catterall             startup/diagnostics, Trace-state reconnect recovery, Servire
                                     operational-log filtering and validation-state lifecycle, and
                                     Fiducia shutdown/PID/log-cleanup behaviour. Added failed
                                     staged-Replay recovery requirement.

  2026-08-30   Nigel       1.6       Defined M0.1 provider/model selection as runtime-global;
               Catterall             documented Praebere-managed stack shutdown semantics; clarified
                                     the single-active Vestigare recording limitation and required
                                     session-binding behaviour when multiple sessions/external
                                     clients exist.

  2026-08-30   Nigel       1.7       Defined Pontis session identity as authoritative for Trace
               Catterall             binding; required explicit active-session selection in
                                     Vestigare; prohibited Trace start with no active session; and
                                     defined M0.1 recording as beginning before the selected
                                     session's first model interaction so a Trace is not silently
                                     incomplete.

  2026-08-30   Nigel       1.8       Refined N9 provider/model lifecycle: startup discovery from
               Catterall             Ollama, optional preferred model, established versus
                                     active-execution sessions, external-client explicit selection,
                                     Rogare dropdown behaviour, and runtime-global model locking
                                     while execution sessions are active.

  2026-09-01   Nigel       1.9       Added the N9.5 Pontis-owned session-management and
               Catterall             orphan-recovery boundary, Rogare lifecycle closure, Praebere
                                     lock release/persistence/reconciliation/reset requirements, and
                                     clarified separation from the Servire authorization heartbeat.

  2026-09-04   Nigel       2.0       Added N9.6.2 deferred-release reconciliation and N9.6.3
               Catterall             external-Ollama, demand-residency and atomic runtime-model
                                     reservation requirements. Removed Praebere ownership of the
                                     Ollama process lifecycle.

  2026-09-05   Nigel       2.1       Clarified Praebere authority for model selection, reservation
               Catterall             and residency; retained Pontis authority for session
                                     identity/lifecycle; defined available, reserved and locked
                                     selection states; and clarified that Pontis `not active` means
                                     not active in model execution, not that the session is closed
                                     or the model is available.

  2026-09-06   Nigel       2.2       Defined the M0.1 cached model-discovery and explicit-refresh
               Catterall             boundary, including stale-cache visibility and failed-load
                                     reservation cancellation.

  2026-09-06   Nigel       2.3       Reconciled successful Vestigare provenance validation; required
               Catterall             authoritative model metadata in `trace_recordings`; defined
                                     Repetere/Fiducia exact recorded-model enforcement under the
                                     runtime-global M0.1 model boundary; and documented Rogare's
                                     manual session-reconnect limitation.

  2026-09-10   Nigel       2.4       Defined staged Replay Experiments as immutable snapshots of
               Catterall             replay-critical source conditions, including the required
                                     model; documented the unstage-and-restage workflow and deferred
                                     cross-model Experiment support.

  2026-09-14   Nigel       2.5       Reconciled the release definition against completed M0.1 Replay
               Catterall /           implementation through Repetere 0.20.18 and Fiducia 0.7.3;
               review                clarified immutable Experiment snapshot authority, final
                                     matched/divergent Replay lifecycle, evidence retention/deletion
                                     semantics, and current implementation sign-off boundary.

  2026-09-15   Nigel       2.6       Reconciled M0.1 with Repetere 0.20.19 per-Run replay-model
               Catterall /           selection and Rogare 0.5.7 model-selection UX. Preserved
               review                immutable source-model provenance while allowing each manual
                                     Replay Run to select its execution model subject to Praebere's
                                     runtime-global reservation boundary; documented Rogare
                                     post-session model discovery, explicit selection,
                                     reserved-model locking and concise user feedback.
  --------------------------------------------------------------------------------------------------

### Revision 2.8 --- 2026-09-22

Reconciled the M0.1 requirements with the implemented and live-proven
runtime-authorization lifecycle and the current release position;
replaced obsolete authorization-heartbeat terminology with periodic
licence authorization; referenced the superseding Runtime Authorization
and Code Protection --- Release Position; updated Praebere/Ollama
provider-health and automatic recovery behaviour; recorded Rogare
terminal provider-loss UI recovery; identified the shipped Rogare
third-party integration as Pi plus pi-acp; and updated
release-gate/completion language so completed authorization work is no
longer presented as outstanding.

### Revision 2.7 --- 2026-09-20

Clean-install Docker baseline, fresh registration/authorization and
Stack startup validated; LAN HTTP/browser-download limitation
documented; final bilateral authorization timing change retained before
release.

## Final M0.1 release reconciliation --- 2026-09-23

The M0.1 release closeout has now been completed against the frozen
Registry-distributed image.

The final distribution path has been live-proven as:

`private Illuminates.One Docker Registry → authenticated pull → fresh Docker volumes → first-start Research Licence acceptance → fresh installation identity → signed registration/authorization → external dependency validation → 83/83 Servire validation checks → managed Stack startup → researcher workflow`.

Final release evidence includes:

-   the frozen Lumen image was removed from the test machine and pulled
    again from `registry.illuminates.one`;
-   all previous M0.1 containers and persistent volumes were removed
    before the final clean-install test;
-   the new installation presented the Research Licence again, proving
    licence-acceptance state was not baked into the image;
-   the installation created fresh local identity/state and completed
    registration and signed authorization successfully;
-   release authorization policy is active at a 24-hour lease, normal
    renewal at approximately 6 hours, and approximately hourly retry
    after temporary authorization-service unavailability;
-   Ollama and MongoDB were validated as available;
-   Servire passed all 83 validation checks and the managed Stack
    started cleanly;
-   the end-to-end researcher workflow, including Rogare, Vestigare,
    Repetere and Fiducia, was exercised successfully;
-   the illustrated `SERVIRE_USER_MANUAL v0.1` has been produced;
-   distribution is through the authenticated private Registry rather
    than researcher-side TAR loading;
-   Registry credentials are per researcher and are separate from Lumen
    installation registration/authorization.

The M0.1 runtime image remains frozen. No further runtime development is
required for release unless a release-blocking defect is discovered.

## Reconciliation update --- 2026-09-22

The runtime authorization work previously retained as an implementation
gate is now **CLOSED for M0.1**. The implemented
Servire/Licentia/Illuminates.One lifecycle has been live-proven,
including persistent Ed25519 installation identity, signed initial
registration and renewal, independent Servire verification,
authorization state transitions and recovery, signed `LOCKED` handling,
and authorization-gated Stack operation.

The clean-state Docker registration/authorization baseline remains
accepted. The authorization-related release configuration is now closed.
The shortened development timings have been replaced by the M0.1 release
policy: a 24-hour authorization lease, normal renewal at approximately 6
hours, and approximately hourly retry following temporary
authorization-service unavailability. The frozen distribution image has
subsequently passed a fresh clean-install test.

Praebere's provider lifecycle has also moved beyond the earlier
startup-or-manual-refresh-only description. M0.1 now detects Ollama
provider loss and recovery automatically, keeps successful
provider-health polling quiet in the operator log, and retains **Refresh
Models** as the explicit operator action for refreshing the discovered
model catalogue. Rogare clears its terminal busy/progress state
immediately when provider loss ends a session.

The current M0.1 runtime authorization/security definition is:

> **Lumen M0.1 Runtime Authorization and Code Protection --- Release
> Position**

That document supersedes the earlier design document of the same
subject.

## Reconciliation update --- 2026-09-20

A clean-state M0.1 Docker installation has now been exercised
successfully. Previous M0.1 containers and persistent volumes were
removed, the distribution was rebuilt without cache, a fresh
installation identity registered with Illuminates.One, signed
authorization completed, and the managed Stack started normally.

The release definition is therefore updated as follows:

-   the basic Docker/bootstrap/registration/authorization/Stack-start
    path is validated;
-   Praebere provider-health behaviour and quiet success-path
    operational logging are accepted;
-   Rogare now clears stale busy/progress UI state immediately when
    provider loss terminates a session;
-   HAProxy host publication has been validated for LAN reachability;
-   the supported M0.1 operator-browser path remains local access
    through `localhost` / `127.0.0.1`;
-   M0.1 does not provide HTTPS for the operator UI, so browsers may
    restrict downloads when Lumen is opened through a plain-HTTP LAN
    address;
-   the M0.1 release authorization policy is active: 24-hour lease,
    approximately 6-hour normal renewal, and approximately hourly retry
    following temporary authorization-service unavailability;
-   final clean-install, workflow and distribution acceptance have
    passed;
-   release documentation, including the illustrated Servire User Manual
    and known limitations, has been produced.

## 1. Purpose

M0.1 is the first defined milestone for the Lumen External Research
Distribution.

Its purpose is to provide a coherent research baseline for observing,
reproducing and repeatedly executing model behaviour under explicit
experimental conditions.

M0.1 is not intended to represent the complete Lumen product or the
eventual commercial distribution.

Aestimare is not part of M0.1.

## 2. Included Lumen Services

The intended M0.1 stack comprises:

-   Rogare;
-   Pontis;
-   Vestigare;
-   Repetere;
-   Fiducia;
-   Moderari;
-   Praebere;
-   Servire;
-   Nuntius;
-   MongoDB as supporting persistence.

The distribution is intended to operate as a Dockerised Lumen stack over
a private Lumen Docker network.

M0.1 is a **single-machine distribution**. All Lumen services,
supporting services and the M0.1 Docker environment must be installed
and operated on the same host machine.

The Dockerised distribution exposes only the host entry points required
for operation and research access:

-   **Pontis:** host port `11435` --- the Lumen entry point for
    supported external clients;
-   **Servire:** host port `11439` --- the operational/control UI entry
    point;
-   **MongoDB:** host port `27018` --- the research/data access entry
    point to Lumen's persisted experimental and Trace data.

MongoDB continues to use its normal container-side port `27017`; the
M0.1 Docker configuration maps host port `27018` to container port
`27017`.

Using `27018` as the host-side MongoDB port reduces the likelihood of
conflict with an existing MongoDB installation already using the
standard host port `27017`. It also allows a researcher to connect
directly to Lumen's MongoDB data with their own preferred database tools
or locally developed research/visualisation tooling.

All other Lumen service communication is intended to remain within the
private Lumen Docker network unless explicitly documented otherwise.

Although Lumen's service architecture may permit components to
communicate across network boundaries, distributed multi-host deployment
is not supported or validated in M0.1.

## 3. Functional Requirements

### 3.1 Model Execution and Trace

M0.1 must:

-   execute model interactions through the Lumen stack;
-   record the effective model execution through Vestigare;
-   preserve the actual context supplied to the model;
-   preserve the effective system prompt;
-   retain the incoming client system prompt as superseded provenance
    when Moderari replaces it, exclude that superseded prompt from
    Replay input, and identify exactly one authoritative effective
    system prompt as replayable execution context;
-   store the authoritative provider and exact model identity used for
    execution in the corresponding Vestigare `trace_recordings`
    document; and
-   retain sufficient provider/model execution provenance to identify
    and enforce the model condition under which the recorded execution
    occurred.

### 3.2 Experiment and Repetition

M0.1 must:

-   represent an Experiment explicitly;
-   associate an Experiment with a source Trace;
-   associate repeated replay executions with that Experiment;
-   allow Fiducia to coordinate repeated executions;
-   allow Repetere to perform each reproduction;
-   retain each resulting execution as a normal Vestigare Trace;
-   display Experiment runs as related evidence.

Each run must expose at least:

-   `MATCHED`;
-   `DIVERGED`; or
-   `FAILED / INCOMPLETE`.

Where existing evidence permits, the first divergence point should be
exposed.

A failed Replay run must remain preserved as experimental evidence while
the staged Experiment remains recoverable. Retrying must create a fresh
isolated Replay session/run and must not overwrite or silently discard
the failed-run evidence.

Staging a source Trace must create an Experiment containing a fixed
snapshot of the replay-critical source conditions available at staging
time, including the authoritative source provider and exact source
model. Subsequent changes to the source `trace_recordings` document must
not silently mutate an already staged Experiment.

The Experiment snapshot remains immutable source provenance. Each manual
Replay Run must separately persist the model selected for that Run as an
execution condition. The default Replay model is the Experiment's
recorded source model, but the researcher may deliberately select
another model from Praebere's available catalogue before starting the
Run. Changing the Run model must not mutate the Experiment, source
Trace, or previous Runs. This permits repeated same-model reproduction
and deliberate cross-model comparison while keeping the changed model
condition explicit.

To incorporate corrected or deliberately changed source metadata in
M0.1, the researcher must unstage the existing Experiment and stage the
source Trace again. Restaging creates a new Experiment snapshot; it must
not rewrite the previous Experiment or its preserved run evidence.

Experiment and Replay evidence use the following durable hierarchy:

``` text
Source Trace
    ↓
Experiment
    ↓
Run
    ↓
Replay-created child Trace
```

Each Run is independent experimental evidence. Retrying or running an
Experiment again must create a new Run and must not overwrite an earlier
Run or child Trace. A Run that fails before recording legitimately may
have no child Trace.

Staging state and evidence retention are separate concerns. Unstaging an
Experiment with historical Runs must retain the Experiment and its
Run/child-Trace evidence. Unstaging an Experiment that has never
produced a Run may remove the empty Experiment. If an already-unstaged
Experiment becomes empty because its final historical Run/child Trace is
explicitly deleted, the now-empty Experiment may also be removed. A
staged Experiment remains available for further execution even if its
last historical child Run is deliberately deleted.

Deleting a replay-created child Trace must remove the corresponding
Vestigare Trace/messages and Run evidence consistently and repair the
parent Experiment's Run references. Explicit destructive Experiment
deletion must remove its replay-created child Traces, Runs and
Experiment definition while retaining the original source Trace.
Destructive deletion must not proceed while the affected execution is
active or cleanup is unsettled.

Repetere reports divergence but does not assess its significance.

Replay has two execution modes:

-   while behaviour continues to match the source Trace, Repetere
    performs deterministic reproduction, compares the live response with
    the recorded response, records matching evidence through the private
    recording path and injects recorded tool results rather than
    re-executing those tools;
-   at the first meaningful divergence, Repetere must persist the fork
    before the divergent response enters the private Vestigare ingestion
    path, stop comparison against the recorded continuation, stop
    recorded tool-result injection and transition to the prepared live
    continuation path;
-   the first divergent interaction must be recorded exactly once
    through the live Vestigare path;
-   divergence is a Run result, not an instruction to terminate
    immediately: the live interaction continues, including real tool
    execution where applicable, until the live turn reaches its normal
    completion boundary;
-   terminal cleanup then closes the Replay recording/session and
    releases the applicable model reservation/residency while preserving
    the Run and child Trace as evidence.

A fully matched Replay terminates when all meaningful source exchanges
have been reproduced and records `MATCHED`. A divergent Replay records
`DIVERGED` only after the live continuation has reached its terminal
boundary.

At staging time, Repetere must read the authoritative provider/model
identity from the source `trace_recordings` document and persist it in
the immutable Experiment snapshot. Repetere must not silently re-read
changed source metadata and thereby mutate that source condition.

At manual Run start, the Run's persisted replay-model selection is
authoritative for execution. The Run defaults to the Experiment's
recorded source model unless the researcher explicitly selects another
available model. Repetere then applies the normal Pontis/Praebere
session-selection lifecycle:

-   Repetere obtains the available model catalogue through
    `\obt praebere models`;
-   the selected Replay model is persisted on the Run before execution;
-   when no runtime-global model is reserved, the selected Replay model
    may be selected/reserved for the Replay session;
-   when the runtime-global model exactly matches the selected Replay
    model, Repetere may reserve/share that model for the Replay session
    and continue normally;
-   when a different runtime-global model is selected, reserved or
    locked, Repetere must log and display a controlled model-conflict
    error and must not start the Replay;
-   when the selected Replay model is unavailable, Repetere must fail
    explicitly before model execution; and
-   Repetere must never load Ollama models directly or silently
    substitute the active model. First-execution activation remains
    Praebere's responsibility.

The resulting Run/child Trace must preserve the effective model actually
used, allowing the researcher to compare Runs of the same immutable
Experiment across different model conditions without altering source
provenance.

Fiducia scheduling remains subject to its current M0.1 compatibility
behaviour. A scheduled execution must not change or interrupt a
conflicting runtime-global model; where its required model conflicts
with the active selection, Fiducia must record a failed scheduled-run
result.

### 3.3 System-Prompt Policy

Moderari must support:

-   `Pass-through`;
-   `Moderari Default`;
-   `Custom`.

`Moderari Default` remains the normal/default behaviour.

Pass-through must genuinely leave the client-supplied system context
unchanged.

Custom mode must allow a researcher-defined prompt to replace the
incoming client system prompt.

### 3.4 Context Compaction

Moderari manages context utilisation during an active session.

When context utilisation exceeds approximately **63% of the available
model context window**, Moderari creates a checkpoint representing a
compacted form of the model's current cognitive/contextual state.

The purpose of compaction is to allow a session to continue beyond the
point at which retaining and repeatedly supplying the complete
accumulated conversation would consume too much of the model's finite
context window. Rather than simply discarding older context, Moderari
attempts to preserve the significant cognitive/contextual state required
for continuity in a more compact representation.

As the session continues, Moderari uses checkpointing to preserve
significant conversational state while reducing the amount of historical
context that must continue to be supplied directly to the model. The
**two most recent checkpoints** are returned as part of the continuing
context.

Context compaction is independent of Moderari's system-prompt policy.

In particular:

-   `Pass-through` means that Moderari does not alter the
    client-supplied system prompt; it does **not** disable Moderari
    context management;
-   `Moderari Default` and `Custom` determine the system-prompt
    condition;
-   context compaction determines how an extended conversation is
    represented after the configured utilisation threshold is reached.

A sufficiently long session may therefore cease to present the complete
original conversational history verbatim and instead present the context
constructed through Moderari's checkpoint/compaction mechanism.

Vestigare must record the effective execution context actually presented
to the model so that checkpointing and compaction remain observable as
part of the Trace evidence.

### 3.5 Saved System Prompts

M0.1 must allow custom system prompts to be:

-   saved under a name;
-   listed;
-   selected for reuse;
-   edited;
-   deleted.

The actual prompt injected into the model remains part of Trace
evidence.

### 3.6 `\obt` Control Commands

M0.1 establishes `\obt` as the common Lumen control-command language.

Pontis must forward `\obt` control traffic to Nuntius rather than send
it through normal ask/answer processing.

Nuntius must:

-   obtain the active service catalogue from Servire;
-   accept an empty catalogue as healthy when Servire is reachable;
-   maintain the available-service catalogue;
-   resolve the configured command owner and route directly to that
    service rather than broadcasting or using legacy consume-or-forward
    traversal;
-   support simple `200`, `204`, and error response semantics.

Services that successfully handle a command return `200`.

A routed owner may return `204` only where the defined operation
legitimately has no response body. Silence, a timeout or the absence of
an authoritative outcome must never be interpreted as success.

Nuntius is an integral Servire-owned control-plane service in M0.1.
Servire starts Nuntius automatically when Servire becomes operational,
before the managed workload stack is started. Stack Start/Stop operates
on the managed workload services rather than defining Nuntius's normal
lifecycle. Servire remains able to start, stop and restart Nuntius
individually and stops Nuntius last during Servire shutdown.

Nuntius must expose its live control-plane diagnostics through its
Nuntius-owned UI surface in Servire. A reconnecting/restarted Rogare
instance must be able to recover the authoritative active Vestigare
Trace state through the Nuntius control path.

### 3.7 Provider and Model Discovery

Praebere must remain the executor and authoritative state owner for
model discovery, selection intent, demand loading and Lumen-owned model
residency.

For M0.1, **Ollama is the only supported and validated model-provider
platform**.

Ollama is external infrastructure and must already be available at the
configured endpoint. The M0.1 default is port `11434`. Praebere must not
start, stop or otherwise assume process ownership of Ollama. If the
configured endpoint is unavailable during Praebere lifecycle startup,
startup must fail explicitly and Servire must roll back the services
started by that stack-start attempt. Alternative model-provider
platforms are outside the supported M0.1 release boundary unless
separately validated and explicitly brought into scope.

On startup, Praebere must query Ollama for the models actually available
locally and persist the resulting model catalogue as authoritative
cached discovery state. Praebere must also monitor provider availability
so that Ollama disappearance and subsequent recovery are detected
automatically without requiring a Stack restart. Successful
provider-health checks are routine and must not flood the
operator-facing log; meaningful availability transitions and failures
remain reportable.

Ordinary model-list queries, model-selection validation, status reads
and Praebere UI polling use the cached model catalogue rather than
repeatedly performing full Ollama model discovery. The Praebere UI
**Refresh Models** action remains the explicit operator mechanism for
replacing that catalogue with current Ollama model discovery. If an
explicit refresh fails, Praebere must retain the last successfully
discovered catalogue and report the refresh failure rather than silently
replacing the catalogue with an empty or partial result.

A configured model name is an optional **preferred model**, not a
requirement that the researcher's Ollama installation contain that
model. If the preferred model is absent, Praebere must remain
operational with no model selected; startup must not fail and another
model must not be silently substituted.

M0.1 distinguishes:

-   **available models** --- models discovered from Ollama;
-   **preferred model** --- optional installation/UI preference;
-   **selected model** --- authoritative runtime-global execution
    selection;
-   **model reservation** --- the runtime-global model atomically
    reserved by Praebere for one or more open Pontis sessions;
-   **active execution session** --- a session whose first model ask has
    activated execution through Praebere;
-   **Praebere-owned residency** --- a resident model that Praebere
    itself requested Ollama to load.

Praebere must support discovery and selection through the common `\obt`
path so that Rogare and supported external clients use the same Lumen
command mechanism.

For M0.1, provider and selected-model state are **runtime-global**, not
session scoped. Model selection and model loading are separate
operations. Selecting a model records intent and atomically reserves
that model in Praebere for the requesting Pontis session; it must not
load the model. Lumen therefore distinguishes:

-   an **established session** --- Pontis has assigned the authoritative
    `session_id`; the session may have no model reservation or may
    reserve the runtime-global selected model;
-   an **active execution session** --- the session has performed its
    first model interaction and its model execution condition has been
    established.

Pontis UI/API text `not active` means **not active in model execution**.
It does not mean that the Pontis session is closed, and it does not by
itself mean that model selection is available. An established,
non-executing session may still reserve the runtime-global model in
Praebere.

Praebere exposes the authoritative model-selection state:

-   **available** --- no open session currently reserves the selected
    model;
-   **reserved** --- one or more open sessions reserve the selected
    model, but none is currently active in model execution;
-   **locked** --- one or more sessions are active in model execution;
    and
-   **reconciliation pending** --- Praebere cannot yet safely determine
    the authoritative reservation/execution state.

The first successful selection reserves the runtime-global model for
that session. Other sessions may reserve the same model. A request for a
different model must be rejected atomically by Praebere while any
reservation exists. If two clients concurrently request different
models, exactly one model may win; both clients must display Praebere's
authoritative confirmed model rather than an optimistic local choice.
Pontis records the confirmed model for routing and presentation but does
not independently create or enforce the model reservation.

Praebere loads the selected model only on the first ask, and only if
Ollama does not already report it resident. Praebere records residency
ownership only when it issued that load. Multiple execution sessions
using the reserved model share the residency. If Ollama cannot load the
selected model, including because it was removed after the last
successful discovery, Praebere must return a controlled model-activation
error, must not register the execution session as active, and must
cancel and persist the requesting session's reservation. The client must
be told that both activation failed and its reservation was cancelled.
Reservations held by other open sessions must remain unchanged; the
resulting selection state is `available` when no reservation remains and
`reserved` when another reservation remains. When the final active
execution session ends, Praebere unloads the model only if it owns that
residency. A model that was already resident before Lumen used it
remains loaded. When the final reserving session ends, Pontis reports
the session-lifecycle change and Praebere releases that session's
reservation; the last selected model may remain recorded as history but
is no longer reserved.

When an external client establishes a session while no model is
reserved, it must be presented with Praebere's available models and
guidance to use:

``` text
\obt praebere model select <model_name>
```

The configured preferred model must **not** be silently selected for an
external client.

Rogare must expose Praebere's authoritative model discovery and
selection through the same `\obt praebere` control path used by external
clients. Model selection occurs only after Pontis has established the
Rogare session.

After session start, Rogare queries `\obt praebere models`. When
selection is available, the Rogare model dropdown must display
Praebere's available model catalogue and allow the researcher to choose
a model explicitly. The **Select Model** action must send the normal
`\obt praebere model select <model_name>` command through Pontis so that
Pontis observes the authoritative Praebere response and records the
confirmed model for the session.

If a runtime-global model is already reserved, Rogare must display only
that authoritative reserved model and disable model choice. After
successful selection, Rogare must refresh the authoritative model state,
collapse/lock the selector to the reserved model, and present concise
conversational confirmation in the form `Model <model_name> selected.`
rather than exposing the raw Pontis/OpenAI-compatible control-response
envelope.

Selecting/reserving a model must not load it. First-execution activation
remains Praebere's responsibility.

On managed stack shutdown, Praebere releases only model residency that
it caused. Lumen must never stop Ollama. Failure to unload Lumen-owned
residency must be reported as degraded partial completion and must not
be concealed by terminating the provider.

If Ollama becomes unavailable while Lumen is running, the failure must
be surfaced through the affected execution path and model/session
lifecycle must converge cleanly. Praebere must detect subsequent
provider recovery automatically. Where provider loss terminally ends a
Rogare session, Rogare must clear its busy/progress state immediately so
that normal session controls do not remain disabled until a tab change
or page refresh.

Independent per-session provider/model execution remains outside the
required M0.1 boundary. Rogare and Repetere may request a model for
their session/Run, but Praebere continues to enforce one runtime-global
reserved model condition: concurrent sessions may share that model and
may not independently execute different models at the same time.

### 3.7.1 Session Management and Model-Lock Release

Pontis is authoritative for session identity and lifecycle. M0.1 must
provide:

-   a Pontis-owned Servire UI showing established and active-execution
    sessions, origin, request/in-flight state, timestamps and Praebere
    registration/release state;
-   graceful End Session and confirmed, authorised Force Close / Clear
    Orphan operations;
-   client-scoped `\obt pontis session status`, `session end` and
    `session new` commands;
-   restricted administrative `\obt pontis sessions`, named
    `session end` and `session force-close` commands;
-   an End Session action in Rogare;
-   termination of the current Rogare session during New Session, normal
    stop and restart; and
-   idempotent, auditable release of the ended session's Praebere
    execution registration through Nuntius.

If Praebere is unavailable, local Pontis closure is partial success: the
session must be shown as `release pending`, and Force Close feedback
must state both the successful local closure and deferred provider
release. On Praebere restart, reconciliation must run automatically,
consume pending releases, update Praebere's active set, acknowledge
Pontis and reach `complete` without requiring another client request.

Ordinary external clients may inspect, end or replace only their own
Pontis session. Administrative cross-session operations must be
authorised. Idle time alone must not classify or close a session as
orphaned. M0.1 does not require Pontis to POST a heartbeat to an
external-client callback address.

Praebere must release its execution lock and any Praebere-owned
residency only after the final **active execution session** has ended.
Praebere must retain the runtime-model reservation until the final
Pontis session reserving that model has ended. Praebere must persist
selected provider/model state, residency ownership and active execution
registrations, restore them fail-safe after restart and reconcile
against Pontis authority. A Pontis session closed while Praebere is
unavailable must remain visibly `release pending`; Praebere restart must
reconcile it, remove it from the active set, acknowledge release to
Pontis and reach `complete`. Safe execution reset is permitted only when
Pontis confirms no active execution sessions remain; forced reset, if
provided, is restricted and auditable emergency recovery.

Periodic Servire licence authorization remains a distinct
licensing/security requirement. It neither proves client-session
liveness nor replaces Pontis session termination.

### 3.8 Trace Separation

Internal `\obt` control traffic must not appear as conversational Trace
turns merely because it passed through Lumen.

If a command changes a model execution condition, the resulting
condition must still be represented in the execution evidence.

Examples include:

-   model;
-   provider;
-   effective system prompt.

Vestigare recording metadata must identify the authoritative provider
and exact model used. Individual request/response bodies may also
contain the model identity, but Repetere and Fiducia must not be
required to decode conversational transport bodies to determine the
source execution model.

### 3.9 Concurrent Session Isolation

M0.1 must be validated with multiple simultaneous Lumen sessions.

Transactions within concurrent sessions must remain bounded to their
originating session. Session-specific conversational context, responses,
Trace evidence and other session-specific state must not leak between
sessions.

For M0.1, conversational state and Moderari system-prompt policy are
**session scoped**.

Live concurrent-session validation with Rogare and Pi demonstrated that
one active session can remain under `Moderari Default` while another
active session is independently changed to `Pass-through`. The change
applies only to the selected/originating session and does not alter the
policy of another active session.

Moderari distinguishes between the **default policy for new sessions**
and the **active policy of each established session**. Changing an
established session's policy does not silently change the new-session
default and does not alter another established session.

Provider/model concurrency remains subject to separate M0.1 validation
and limitations; the demonstrated session-scoped system-prompt behaviour
must not be generalized into an unsupported provider/model claim.

### 3.10 Client and Tool Responsibility

M0.1 supports two distinct client/tool responsibility paths.

#### External Clients

When an external client is used, that client is responsible for its own
tool environment.

This includes responsibility for:

-   declaring or exposing the tools available to the model;
-   providing any tool definitions required by the client/model
    interaction;
-   executing tool calls;
-   returning tool results into the continuing client/model interaction.

Lumen does not prescribe or manage an external client's tool environment
in M0.1.

From Lumen's perspective, client-managed tools form part of the external
client's execution environment.

M0.1 therefore makes no claim of compatibility with, or responsibility
for, arbitrary external-client tool implementations.

#### Rogare

Rogare has no native tool environment.

Where tool use is required, Rogare relies upon an external tool
provider.

For M0.1:

-   Rogare's supported and validated third-party integration uses Pi
    together with `pi-acp`;
-   the frozen M0.1 distribution includes the applicable Pi and `pi-acp`
    components and their third-party licence notices;
-   Rogare uses the existing Lumen integration to access that tool path;
    and
-   alternative tool providers are not supported through Rogare in M0.1.

Broader Rogare tool-provider discovery, configurable provider locations,
multiple simultaneous providers and alternative tool-provider
integrations remain outside the M0.1 release boundary unless separately
validated and brought into scope.

### 3.11 Runtime Authorization and Code Protection

M0.1 implements the runtime authorization boundary defined by **Lumen
M0.1 Runtime Authorization and Code Protection --- Release Position**.

For the external research distribution:

-   possession of the Lumen M0.1 distribution does not, by itself,
    authorize operation of a Lumen installation;
-   each installation has a persistent cryptographic installation
    identity;
-   the implemented installation identity uses Ed25519;
-   the installation private key remains local to the Lumen
    installation;
-   Illuminates.One is the authorization authority;
-   Licentia is the authorization transport/protocol broker and does not
    make authorization decisions;
-   Servire signs authorization requests using the installation identity
    and independently verifies Illuminates.One-signed authorization
    decisions;
-   successful initial registration binds the installation
    identity/public key to its registration at Illuminates.One;
-   continued operation requires periodic licence authorization with
    Illuminates.One;
-   authorization uses a time-limited lease and bounded failure/recovery
    lifecycle rather than requiring continuous connectivity;
-   the shortened development/test lease, renewal and retry values must
    be replaced consistently on both Lumen and Illuminates.One with the
    agreed M0.1 release values before the final distribution image is
    frozen;
-   Servire is the local runtime security gate and supported operational
    authority for the managed Lumen Stack;
-   individual Lumen services do not independently implement the
    external Illuminates.One authorization relationship;
-   direct execution of individual service internals is not a supported
    researcher runtime path;
-   ordinary operational `config.yaml` is not required to be encrypted
    at rest for M0.1; cryptographic secrets are protected separately;
-   the distributed installation contains no Illuminates.One private
    signing key, permanent authorization bypass or equivalent master
    authorization secret;
-   copying the distribution, container or configuration does not, by
    itself, create another authorized Lumen installation; and
-   M0.1 does not claim absolute tamper resistance against a determined
    operator who controls the host on which the software executes.

Licence authorization communicates technical information required to
register, identify and validate the Lumen installation. It does not
transmit prompts, model responses, conversations, Trace or Replay
contents, research results, user documents, datasets or other
substantive research content as part of licence authorization.

The detailed implemented release boundary, trust ownership,
authorization lifecycle, protection objectives and limitations are
defined in
`LUMEN_M0.1_RUNTIME_AUTHORIZATION_AND_CODE_PROTECTION_RELEASE_POSITION.md`
rather than duplicated here.

## 4. Research Requirements

M0.1 must allow experimental conditions to be explicit enough that
repeated executions can be meaningfully compared.

The release must preserve the principle:

> **Experimental conditions should be explicit, reproducible and
> observable. Lumen should record what happened without unnecessarily
> altering the conditions being investigated.**

M0.1 provides evidence and objective replay-divergence information.

It does not provide behavioural assessment.

## 5. Known Limitations

### 5.1 No Aestimare

Aestimare is explicitly excluded.

M0.1 does not determine whether a divergence is significant, desirable,
undesirable, trustworthy or untrustworthy.

### 5.2 No Human Response Rating

Rogare does not yet collect independent 1--10 correctness and quality
ratings from researchers.

This is future work intended eventually to provide human assessment
evidence for comparison with Aestimare.

### 5.3 Single Lumen Tool Provider Assumption

The Lumen-managed tool path may remain limited to one attached tool
provider in M0.1.

For Rogare, the supported and validated integration is the distributed
Pi plus `pi-acp` tool path.

This limitation does not apply to tool environments owned and executed
by an external client. Those remain the responsibility of that client
and are outside Lumen's M0.1 tool-provider compatibility claim.

The architecture does not define the Lumen-managed tool path as
permanently single-provider.

Multiple providers, provider discovery, namespaces and a unified tool
catalogue remain future development.

### 5.4 No Resource-Bound Behaviour Experimentation Feature

M0.1 does not provide dedicated facilities for controlled experiments
varying computational resource envelopes.

This remains a research question rather than an M0.1 feature.

### 5.5 No General Multi-Operator Servire Access

M0.1's successful multi-session validation does not imply that the
Research Distribution is intended for unrestricted simultaneous use by
multiple researchers.

Before general external research release, Servire is expected to enforce
a single active operator session per installation.

That enforcement is not an M0.1 requirement.

### 5.6 Concurrent Execution Conditions

Moderari system-prompt policy is no longer a shared global execution
condition.

M0.1 now supports independently established **session-scoped Moderari
system-prompt policy**. Concurrent live validation demonstrated a Rogare
session remaining under `Moderari Default` while a Pi/external-client
session was changed to `Pass-through`. Policy status queries from each
session reported its own state, and Servire displayed the selected
session's policy independently from the default used for new sessions.

The following distinctions therefore apply:

-   **new-session default** --- the policy inherited when a new Moderari
    session is established;
-   **session active policy** --- the policy currently applied to model
    requests for that established session;
-   changing one session's active policy does not change another
    session;
-   changing one session's active policy does not change the new-session
    default.

Provider/model selection is deliberately **runtime-global for M0.1**.
Concurrent sessions do not receive independent model selections.
Praebere atomically reserves the first successfully selected model for
the selecting Pontis session before execution begins. Other sessions may
join that same reservation but cannot reserve a different model until
every reserving session has ended. First ask activates execution and
demand-loads the reserved model; it does not decide which client wins
model selection. This must remain distinct from Moderari system-prompt
policy, which is session scoped.

### 5.7 Model Discovery Is Cached; Provider Health Is Monitored

Praebere's model catalogue is a snapshot from startup or the most recent
successful explicit **Refresh Models** action. Provider availability is
monitored automatically, but provider-health recovery is distinct from a
full model-catalogue refresh. Models added to or removed from Ollama
outside Lumen are therefore not necessarily reflected in ordinary
`\obt praebere models` responses or the Praebere UI until the researcher
uses **Refresh Models** or restarts the Stack.

A model appearing as `available` in Praebere means that it existed at
the last successful discovery; it does not guarantee that the model
still exists or can currently be loaded. If Ollama cannot load a cached
model on first ask, the ask fails with a controlled activation error and
the requesting session's reservation is cancelled. The researcher must
correct the Ollama condition or refresh the catalogue before selecting
again.

If live refresh cannot reach Ollama or otherwise fails, Praebere
deliberately retains the previous catalogue and displays the refresh
failure. This fail-safe behaviour avoids silently discarding the last
known discovery state, but the retained catalogue may be stale until a
later refresh succeeds.

### 5.8 Nuntius Is Lightweight

Nuntius is not a durable message broker.

M0.1 does not require:

-   RabbitMQ;
-   Kafka;
-   guaranteed delivery;
-   durable command queues;
-   distributed consensus;
-   sophisticated subscriptions.

### 5.9 Saved Prompt Management Is Minimal

M0.1 does not require:

-   prompt sharing;
-   organisational prompt libraries;
-   prompt permissions;
-   prompt version history;
-   collaborative editing.

### 5.10 Mid-Session System-Prompt Policy Changes

Moderari system-prompt policy is evaluated at model-request time rather
than being fixed for the lifetime of an external-client session.

Current testing with Pi has demonstrated that an active session can
transition between `Moderari Default` and `Pass-through` without
restarting the client session. Concurrent Rogare/Pi testing additionally
demonstrated that this transition is session scoped: changing the Pi
session to `Pass-through` left the already-active Rogare session on
`Moderari Default`, while the default for newly created sessions also
remained `Moderari Default`. A subsequent request is processed using the
newly selected policy while the existing conversational history remains
available. For example, a session may begin under `Moderari Default`,
change to `Pass-through`, and later return to `Moderari Default`, with
the model retaining conversational continuity across those transitions.

This is valid and potentially useful research behaviour, but it creates
an important Trace and Replay consideration. A single recorded session
may contain model executions produced under different system-prompt
policies and therefore under different effective system prompts. The
policy transition is part of the experimental condition and must not be
treated as though one system prompt governed the entire session.

In particular, an external client may continue to carry its own system
message in the conversation while Moderari later applies
`Moderari Default` or `Custom`. Depending on how Vestigare records the
incoming client context and the effective model context, a Trace may
therefore contain evidence of both the client-supplied system prompt and
a Moderari-applied system prompt.

This has implications for Repetere. Replay must not assume that every
recorded client system message should always be retained, nor that every
client system message should always be suppressed when a Moderari prompt
is present. If Vestigare records the external client's system prompt and
also records Moderari's effective replacement, Repetere requires
sufficient provenance to distinguish:

-   the system context supplied by the external client;
-   the system-prompt policy active for each model execution;
-   the effective system prompt actually supplied to the model; and
-   any policy transition occurring during the recorded session.

Without that distinction, Replay could incorrectly send both the
external-client system prompt and the Moderari-generated prompt, or
incorrectly remove a client system prompt that was intentionally active
during a `Pass-through` portion of the session.

Vestigare now records authoritative system-prompt provenance for each
applicable exchange. The incoming client prompt is retained with
`disposition: superseded` and is not Replay input when Moderari replaces
it. The Lumen/Moderari effective system prompt is retained separately
and identified as the single replayable system prompt. Replay must
continue to use this explicit provenance rather than infer behaviour
solely from message role or position.

For research use, changing the policy mid-session should be regarded as
an explicit change in experimental conditions. A researcher seeking a
single controlled system-prompt condition across an experiment should
avoid changing the policy during the recorded session. A researcher
intentionally investigating system-prompt transitions may use this
behaviour, provided the resulting Trace preserves enough evidence to
reproduce the transition correctly.

Rogare does not itself supply a client system prompt. When Rogare is
used as the client, the effective system prompt is therefore established
through Moderari, but the same requirement remains: any policy change
during a recorded session must be represented in the execution evidence.

### 5.11 Context Compaction Is Part of the Experimental Condition

Moderari's context-compaction mechanism is active in M0.1 and may
materially affect model behaviour once the configured utilisation
threshold is reached.

Whether compaction is desirable, neutral or detrimental is not
predetermined by M0.1. Its effect may depend upon the model,
context-window size, conversation, checkpoint contents and experimental
objective.

Researchers should therefore treat context compaction as an observable
execution condition rather than assume that a compacted session is
behaviourally equivalent to the complete un-compacted conversation.

M0.1 may itself be used to investigate this behaviour, including
comparison of executions before and after compaction and examination of
whether checkpointing successfully preserves sufficient
cognitive/contextual state.

`Pass-through` does not disable context compaction. It controls
treatment of client-supplied system prompts, not Moderari's
context-management behaviour.

### 5.12 Servire Operations Log Filtering

Servire's Operations Log is an operator-facing view rather than a
complete copy of every service access log.

Routine successful internal `/api/...` polling/access traffic is
filtered from the consolidated Operations Log so that model/service
polling does not obscure meaningful operator and lifecycle activity.
Service-owned detailed logs remain authoritative and are not removed by
this presentation filter.

Meaningful control-plane lifecycle events, warnings, errors and
unsuccessful API responses remain visible. Nuntius command-routing and
transport diagnostics remain available through the Nuntius-owned
diagnostics UI exposed within Servire.

### 5.13 Replay Model Selection Uses the Runtime-Global Reservation Boundary

M0.1 does not provide private simultaneous model residency for
individual Repetere Runs. It does, however, allow each manual Replay Run
to select its execution model.

The Experiment retains the immutable authoritative provider/model
identity captured from its source Trace. That source model is the
default for a new Run and remains the reference condition. Selecting a
different Replay model creates a changed execution condition on that
Run; it does not rewrite the Experiment or source Trace.

At manual Replay Run start:

-   Repetere obtains Praebere's available model catalogue through
    `\obt praebere models`;
-   the researcher may retain the source model or explicitly select
    another available model for the Run;
-   the selected Replay model is persisted with the Run;
-   if no model is currently reserved, the Run may reserve its selected
    model through the ordinary Pontis/Praebere lifecycle;
-   if the runtime-global reserved model matches the Run's selected
    model, Replay may share it as another normal session;
-   if a different model is already reserved or locked, Replay must stop
    with a clear conflict rather than substitute that model or disturb
    active/reserving sessions; and
-   model activation remains lazy and occurs on first execution through
    Praebere.

Consequently, researchers can run the same immutable Experiment against
different models sequentially and retain separate Run/child-Trace
evidence for comparison. M0.1 still does not permit concurrent Runs or
sessions to execute different models under independent per-session model
residency.

Editing source Trace metadata after staging does not change an existing
Experiment. To change the captured source condition itself, the
researcher must unstage/restage as defined by the Experiment lifecycle.
This is distinct from deliberately selecting a different Replay model
for an individual Run.

### 5.14 Single-Machine Deployment

M0.1 must be installed and operated on a single host machine.

Researchers must not distribute individual Lumen services across
multiple machines and expect that topology to represent a supported M0.1
research environment.

Multi-host/distributed deployment remains future development and
requires separate validation of service discovery, transport,
operational control, security, authorization and logging.

### 5.15 Research Distribution Is Not the Commercial Product

M0.1 does not define:

-   commercial licensing;
-   commercial multi-user entitlements;
-   commercial operator limits;
-   Aestimare assessment services;
-   the final deployment/support model.

### 5.16 Runtime Protection Is Not Absolute Tamper Resistance

M0.1 runtime authorization and distribution protection are intended to
prevent accidental unauthorised use, simple copying and ordinary
operation outside the authorised installation model.

They do **not** make source code or locally executing software
impossible for the machine owner to inspect, modify or patch.

A sufficiently determined operator with source-code and machine access
may ultimately be able to bypass local checks or extract locally held
material.

The security objective is therefore to make circumvention deliberate,
preserve installation provenance, prevent simple redistribution from
producing a functioning authorised installation, and reduce reliance on
contractual enforcement. It must not be represented as absolute DRM or a
hardware trust boundary.

### 5.17 Runtime Authorization Requires Periodic Illuminates.One Availability

Normal external M0.1 operation depends on periodically renewing
authorization with Illuminates.One.

Temporary connectivity failure must be tolerated through the
authorization lease/grace model, but the installation is not intended to
operate indefinitely offline.

Offline research licensing, hardware migration, key rotation, capability
manifests and the complete commercial licensing system remain future
work unless separately brought into M0.1.

### 5.18 Single Active Vestigare Recording

M0.1 supports only **one active Vestigare recording at a time** across
the Lumen installation, regardless of how many Rogare or external-client
sessions are active.

Pontis is authoritative for Lumen session identity. Every supported
session, including an otherwise unnamed external-client session, is
assigned a Pontis `session_id`, and session-correlated traffic
traversing the model path carries that identity and its associated
session metadata.

Vestigare must never infer recording ownership from the "last-opened",
"most recent" or "currently busiest" session. An active Trace is bound
explicitly to one Pontis `session_id`:

``` text
trace_id -> Pontis session_id
```

Traffic for other concurrent sessions continues normally but must not be
incorporated into the active Trace.

#### Trace Start and Session Selection

A Trace cannot be started when there are **no active Lumen sessions**.
The recording control must be disabled or the request rejected clearly
because there is no session to which the Trace can be bound.

When exactly one eligible active session exists, that session may be
selected automatically, but the UI must display the actual Pontis
session identity being recorded.

When more than one eligible active session exists, Vestigare must
present the active Pontis sessions and require the researcher to select
which session is to be recorded. M0.1 must not silently choose a
session.

Where Trace Start originates from a session-aware control path, the
originating Pontis `session_id` may be supplied as the proposed
selection, but the resulting recording must still persist that explicit
session binding.

Once a Trace is active, attempts to start another recording must be
prevented.

#### Recording Start Boundary

For M0.1, a research Trace intended to represent a complete session must
begin **before the selected session's first model interaction**.

Starting Vestigare after a session has already exchanged model traffic
would otherwise miss the earlier execution sequence. A later request may
carry conversational context containing earlier messages, but that is
not equivalent to having observed and recorded the earlier requests,
responses, tool activity, execution conditions and other provenance as
they occurred.

M0.1 must therefore not silently represent a mid-session recording as a
complete session Trace. Trace Start is permitted only for an eligible
active session that has not yet performed its first model interaction.

This supports the normal sequence: the client connects, Pontis
establishes and names the session, the researcher starts Vestigare for
that session, and only then submits the first research prompt.

Explicit partial/mid-session Trace support, historical backfilling, and
multiple simultaneous Vestigare recordings are outside the required M0.1
boundary.

**M0.1 acceptance:** create at least two simultaneous eligible sessions,
confirm that Vestigare presents both Pontis session identities for
selection, select one session, start recording before its first model
interaction, send traffic through both sessions, and confirm that only
the bound session appears in the Trace. Also verify that Trace Start is
unavailable with no active session and is rejected for a session that
has already begun model interaction.

### 5.19 Local Browser Access, LAN HTTP and Log Downloads

M0.1 is a **single-machine research distribution** and does not provide
HTTPS for the operator-facing browser interfaces.

The supported M0.1 browser-access path is therefore local access on the
machine running Lumen, using `http://localhost:<port>` or
`http://127.0.0.1:<port>` as applicable.

HAProxy may publish Lumen entry points on the host's LAN interfaces, and
LAN reachability has been validated. However, plain-HTTP LAN access is
not claimed as a fully supported secure browser experience in M0.1.
Modern browsers may apply additional restrictions to downloads from an
insecure non-loopback origin. In Chrome, this has been observed to block
a Servire operational-log download when Servire is opened through a
`http://192.168.x.x:<port>` address even though the generated log
content is valid.

Servire operational-log export has been validated through localhost. The
LAN behaviour is a browser security restriction, not a failure of
Servire log generation.

HTTPS/TLS for operator-facing remote browser access remains post-M0.1
work.

### 5.20 Rogare Session Reconnect Is Manual

Rogare does not automatically discover or reconnect to an existing
Pontis session in M0.1. After Rogare restart or disconnection, the
researcher must obtain the active session ID from Pontis, enter it in
Rogare's Session field and start using that session.

The authoritative Pontis session and its model/conversational context
continue, but Rogare does not restore its prior local conversation
display. Earlier messages may therefore be absent from the Rogare UI
even though the continuing model request retains the established Lumen
session context.

Automatic session discovery and restoration of Rogare conversation
presentation remain future improvements. This limitation does not
indicate loss of Pontis session identity or model context.

## 6. Pre-General-Research-Release Requirement

Before the Research Distribution is made generally available to external
researchers, Servire should enforce a single active operator session per
installation.

The mechanism should:

-   permit legitimate remote browser operation;
-   decline a second simultaneous operator cleanly;
-   explain that an active session already exists;
-   provide an explicit Exit/Release Session action;
-   use a renewable lease/timeout so abandoned sessions do not
    permanently lock the installation.

This is a distribution/access policy and must not be confused with
Lumen's underlying ability to isolate concurrent sessions.

## 6.1 Implementation Status --- 2026-09-22

The core functional development represented by the M0.1 N1--N10
development chain remains **development-signed-off**. Replay acceptance
has subsequently been closed for M0.1, with deferred negative/recovery
hardening retained as post-M0.1 work rather than a release blocker.

The Servire/Licentia/Illuminates.One runtime-authorization lifecycle is
also **CLOSED for M0.1** and has been implemented and live-proven. This
includes persistent Ed25519 installation identity, signed initial
registration and renewal, independent Servire verification,
authorization lifecycle and recovery, signed `LOCKED` handling,
authorization-gated Stack operation and the Servire authorization UI.

A clean Docker installation with previous M0.1 containers and persistent
volumes removed has also been rebuilt without cache, registered as a
fresh installation, authorized successfully and used to start the
managed Stack.

Release closeout is complete for the M0.1 baseline. The frozen Registry
image has been re-pulled onto a clean test installation with prior
containers, volumes and cached Lumen image removed; first-start licence
acceptance, fresh registration/authorization, dependency validation,
83/83 Servire validation checks and managed Stack startup all passed.
The illustrated Servire User Manual has also been produced.

## 7. M0.1 Completion Criteria

M0.1 can be considered complete when:

-   the Experiment structure works end-to-end;
-   Fiducia coordinates repeated Experiment runs;
-   Repetere divergence is visible;
-   Moderari system-prompt policy is configurable;
-   saved custom prompts can be reused;
-   external-client tool declaration and execution remain the
    responsibility of the external client;
-   Rogare tool use is validated against the supported Pi provider in
    its expected/default installation location;
-   Nuntius provides the common `\obt` distribution path;
-   Praebere discovery/selection uses the agreed control path;
-   `\obt` traffic remains outside conversational Trace;
-   effective execution context remains observable;
-   provider/model Trace provenance and any Replay binding behaviour
    have been established and documented before a provider/model Replay
    guarantee is made;
-   every new Vestigare `trace_recordings` document records the
    authoritative provider and exact model used for execution;
-   Repetere defaults each new Run to the Experiment's recorded source
    model, permits explicit per-Run selection of another
    Praebere-available model, persists that execution condition on the
    Run, and proceeds only when the runtime-global reservation is
    compatible;
-   Fiducia records a failed scheduled-run result instead of changing
    the global model when the scheduled Trace model conflicts with the
    active selection;
-   Moderari checkpoint/context-compaction behaviour remains observable
    in Trace evidence;
-   observed mid-session system-prompt policy switching and its
    Trace/Replay provenance implications are documented;
-   concurrent-session isolation has been demonstrated, including
    independent Moderari system-prompt policy state for simultaneous
    Rogare and Pi sessions and separation of established-session state
    from the new-session default;
-   Vestigare enforces the M0.1 single-active-recording restriction and
    binds the active Trace to one authoritative Pontis `session_id`;
-   Trace Start is unavailable when no active session exists;
-   Vestigare presents active Pontis sessions for explicit selection
    when more than one eligible session exists and never guesses from
    recency/activity;
-   M0.1 complete-session recording can begin only before the selected
    session's first model interaction;
-   a multi-client Trace validation proves traffic from unbound sessions
    is excluded and the selected session identity is persisted in the
    Trace;
-   a Praebere restart validation proves deferred Pontis releases
    reconcile automatically and reconciliation reaches `complete`
    without a further client ask;
-   Force Close reports local closure plus deferred Praebere release as
    partial success;
-   Praebere startup fails explicitly when the configured Ollama
    endpoint is unavailable, causing Servire to roll back that
    stack-start attempt;
-   selecting a model does not load it; the first ask demand-loads it
    when absent;
-   Praebere atomically enforces one runtime-global model reservation
    while allowing concurrent sessions to share the same reserved model;
-   clients display the Praebere-confirmed authoritative model after
    selection success or conflict, including the current reserved model
    in human-readable conflict feedback;
-   after a Rogare session is established, Rogare displays Praebere's
    available models when selection is available, sends explicit
    selection through Pontis, shows only and disables the authoritative
    reserved model when a reservation exists, and confirms a successful
    selection as `Model <model_name> selected.` without exposing the raw
    control-response envelope;
-   stopping the managed stack causes Praebere to unload only model
    residency that it caused and never stops externally operated Ollama;
-   the complete stack passes integration/regression validation as a
    single-host installation;
-   installation identity and signed runtime authorization work
    end-to-end **\[validated\]**;
-   a clean Docker installation with prior M0.1 containers/volumes
    removed creates a fresh installation identity, completes initial
    registration and signed authorization, and starts the managed Stack
    successfully **\[validated\]**;
-   authorization lease renewal, expiry/recovery and signed `LOCKED`
    behaviour have been demonstrated **\[validated\]**;
-   Servire gates managed Stack operation according to the implemented
    authorization lifecycle **\[validated\]**;
-   the final non-test lease/renewal/retry values are configured
    consistently on both Lumen and Illuminates.One **\[validated: 24h /
    \~6h / \~1h\]**;
-   copying/re-pulling the frozen distribution does not, by itself,
    constitute or create another authorized Lumen installation
    **\[validated by clean-volume, clean-image re-pull and fresh
    registration\]**;
-   the frozen distribution is accepted against the documented M0.1
    security boundary: installation identity and licence acceptance are
    installation state rather than baked image state, and
    Illuminates.One signing authority is not part of the supported
    distribution **\[validated release boundary\]**;
-   the frozen Research Licence and applicable Pi and `pi-acp`
    third-party licence notices are included in the release;
-   known limitations are documented.

## 8. Release Boundary

M0.1 should remain deliberately bounded.

Features should not be added merely because they are desirable for the
eventual Lumen ecosystem.

The milestone is complete when it provides a stable research baseline
for:

> **Observe → Reproduce → Repeat**

with explicit experimental conditions and sufficient provenance to
support later assessment.

Aestimare and broader Reasoning Assurance capabilities build upon that
evidence; they do not need to be pulled into M0.1.
