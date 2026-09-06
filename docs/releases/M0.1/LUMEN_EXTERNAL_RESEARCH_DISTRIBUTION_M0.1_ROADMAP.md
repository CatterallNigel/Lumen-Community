# Lumen External Research Distribution M0.1 --- Development Roadmap

**Date:** 2026-08-22\
**Status:** Active M0.1 roadmap — N1–N9 and N10 live validation closed; remaining release work retained below

> **"Plans are worthless, but planning is everything."**\
> --- Dwight D. Eisenhower

## Document Revision History

| Date       | By              | Version | Description                                                                                                                                                                                                               |
| ---------- | --------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-08-23 | Nigel Catterall | 1.0     | First reviewed release.                                                                                                                                                                                                   |
| 2026-08-25 | Nigel Catterall | 1.1     | Added Fiducia stale PID-file recovery requirement and acceptance validation.                                                                                                                                              |
| 2026-08-26 | Nigel Catterall | 1.2     | Reconciled N1–N6 implementation and live validation; recorded Trace ownership/session findings, system-prompt Trace finding, current bugs and optional M0.1 polish.                                                       |
| 2026-08-26 | Nigel Catterall | 1.3     | Added future Servire HTTPS/TLS operator-interface requirement to the post-M0.1 development roadmap.                                                                                                                       |
| 2026-08-28 | Nigel Catterall | 1.4     | Reconciled N7 Moderari implementation and live concurrent-session validation; system-prompt policy is session scoped.                                                                                                     |
| 2026-08-30 | Nigel Catterall | 1.5     | Reconciled N7+ Moderari, saved prompts, effective-prompt provenance, repeated context compaction and Repetere C/D/E progress.                                                                                             |
| 2026-08-30 | Nigel Catterall | 1.6     | Reconciled N8 diagnostics/UI and Servire control-plane integration progress; added Repetere failed staged-Replay recovery lifecycle work.                                                                                 |
| 2026-08-30 | Nigel Catterall | 1.7     | Defined M0.1 provider/model selection as runtime-global; per-session provider/model selection remains optional polish/future development.                                                                                 |
| 2026-08-30 | Nigel Catterall | 1.8     | Added Vestigare active-session selection and complete-session recording boundary: Pontis session identity is authoritative; Trace Start requires an eligible active session and must precede its first model interaction. |
| 2026-08-30 | Nigel Catterall | 1.9     | Refined N9 model lifecycle: Praebere startup discovery, preferred-versus-selected model, established versus active-execution sessions and runtime-global model locking while execution sessions exist.                    |
| 2026-09-01 | Nigel Catterall | 2.0     | Added N9.5 Pontis-owned session management, Rogare/external-client explicit closure, authorised orphan cleanup, Praebere lock release and restart reconciliation requirements.                                            |
| 2026-09-06 | Nigel Catterall | 2.1     | Reconciled the reviewed N6–N10/Praebere document chain; closed N8 diagnostics, N9 and N10 live compatibility validation; replaced the stale Phase 6 checklist with the implemented runtime-global lifecycle; and added M0.1 trace-model recording and replay-start model enforcement. |

## Purpose

This roadmap orders the work currently identified as necessary to reach
the Lumen External Research Distribution M0.1 milestone.

It is intentionally an engineering sequence rather than a calendar
commitment.

M0.1 is intended to establish a coherent, reproducible research
baseline. The sequence below is dependency-driven: the shared control
plane is established before the M0.1 features that consume it.

All M0.1 development and acceptance validation assumes a **single-host
installation**. Distributed multi-host deployment is outside the M0.1
release boundary.

### Progress notation

- `[x]` --- implemented/validated sufficiently for the current development stage.
- `[ ]` --- still required, blocked, or reserved for later formal acceptance.
- A checked development item may still be repeated during Phase 12 formal release-candidate acceptance.

## Phase 1 --- Confirm Existing Behaviour

Before changing command or replay behaviour, establish the current
implementation baseline.

-   [x] Inspect existing Repetere `\obt` handling and injection. **Completed during N6.**
-   [x] Inspect existing Moderari `\obt` handling and injection. **Completed during N7.**
-   [x] Document exactly how Moderari extracts, removes, rewrites or reinserts
    tool-related information from an incoming client system prompt.
    **Completed during N7+: Default, Pass-through and exact Custom behaviour are explicit.**
-   [x] Confirm the effective Moderari Default system prompt currently
    presented to the model when tools are available. **Observed in live
    execution/logging; Trace fidelity is a separate finding below.**
-   [x] Inspect Pontis recognition/routing of `\obt`. **N5 completed and live validated.**
-   [x] Confirm current Praebere provider/model discovery and selection
    paths. **Completed during N9.1 current-state audit.**
-   [x] Confirm current Trace representation of system prompts and Replay executions.
    **N7+ corrected the effective-prompt evidence boundary. Provider/model provenance
    remains separately tracked for Praebere/N9.**
-   [x] Establish where provider/model identity appears in Trace evidence.
    **The exact model is present in recorded request/response messages, but recording-level
    model metadata is absent; the required M0.1 correction is retained in Phase 5.**
-   [x] Determine current Repetere provider/model behaviour. **Repetere does not yet
    enforce the source Trace model at replay start; the required M0.1 correction is
    retained in Phase 5.**
-   [x] Determine whether provider/model identity appears indirectly in conversational
    evidence. **It appears in transport bodies and must instead be available explicitly
    from authoritative recording metadata.**
-   [x] Resolve the M0.1 different-model replay boundary. **A replay must not substitute
    a different active global model. Concurrent replay-private models remain deferred
    until general per-session model selection.**
-   [x] Compare baseline/Default, `Pass-through`, and `Custom` executions.
    **Completed during N7+: incoming provenance and the effective system prompt
    are distinguishable, with the effective prompt authoritative for execution evidence.**
-   [x] Determine which system-prompt record Repetere reconstructs and whether
    Replay can send two active system prompts. **Repetere C investigation completed:
    Replay reconstructs one authoritative effective system prompt.**
-   [x] Confirm current Servire knowledge of configured/running services.
    **Validated through the Nuntius catalogue and live service-state updates.**
-   [x] Confirm how request/session correlation currently works for control
    responses. **Validated through Pontis → Nuntius → Repetere and both
    Rogare/external-client paths.**
-   [x] Inspect current Repetere session creation/reuse behaviour.
    **Completed during the N6+ Replay-fidelity pass.**
-   [x] Determine whether sequential Replay runs create fresh sessions or inherit
    prior context/state. **Fresh Replay-session isolation is established for the current path.**
-   [x] Confirm Moderari context-compaction behaviour at the approximately 63%
    utilisation threshold and its observable post-compaction context.
    **120K-file validation produced two rolling compactions (71.50% → 34.09% and
    69.07% → 30.96%) followed by normal terminal completion. See
    `MODERARI_CONTEXT_COMPACTION_120K_VALIDATION_EVIDENCE_2026-08-29.md`.**

**Current finding:** the baseline investigation is no longer purely
pre-development. N5/N6 and the Trace/session work have established several
integration boundaries, while the remaining Moderari-, Praebere-, Replay-
and compaction-specific questions remain explicit work rather than implicit
assumptions.

**Exit condition:** existing behaviour and integration points are understood
well enough to change them without introducing parallel mechanisms,
including a documented understanding of whether provider/model identity is
Trace provenance, Replay-binding information, or both.

## Phase 2 --- Servire Control-Plane Configuration

Extend Servire's existing service descriptions so they can define the
active `\obt` topology required by Nuntius.

-   [x] Add/configure `obt_enabled` per service.
-   [x] Define command ownership metadata.
-   [x] Define successful query-response targets.
-   [x] Extend `\obt services` so Nuntius can obtain the active
    control-plane catalogue.
-   [x] Accept a valid empty active catalogue.
-   [x] Define runtime service-start/service-stop catalogue updates.
-   [x] Validate duplicate/conflicting exclusive command ownership.

Do not add `log_class` or operational-log user/internal classification
in this phase.

**Status 2026-08-26:** Phase 2 is complete for the current Nuntius control-plane
implementation. Live Nuntius startup demonstrated catalogue growth as Servire
started managed services.

**Exit condition:** Servire can authoritatively describe which active service
owns each configured control command and where successful query results must
be delivered.

## Phase 3 --- Nuntius Control Plane

Implement Nuntius against the Servire-defined topology.

-   [x] Implement Nuntius service skeleton and health.
-   [x] Bootstrap directly from Servire using `\obt services`.
-   [x] Treat a successful empty catalogue as healthy.
-   [x] Build an in-memory routing dictionary.
-   [x] Keep the dictionary current from Servire topology updates.
-   [x] Standardise the service `\obt` endpoint for migrated services.
-   [x] Implement `204`, `200` with/without body, and `4xx/5xx` response semantics.
-   [x] Route commands directly to configured owners rather than broadcasting normally.
-   [x] Preserve request correlation while commands execute.
-   [x] Return query responses to Pontis.
-   [x] Deliver response bodies only to additional consumers configured by Servire.
-   [x] Verify control traffic remains outside the ask/answer Trace path.
-   [x] Implement Nuntius diagnostic logging for command routing, full exchanged
    control messages, responses, failures and timeouts.
-   [x] Expose the completed Nuntius diagnostics through the Nuntius UI within
    Servire. **Completed during N8.1/N8.2; the Nuntius-owned diagnostics surface is
    available directly and through the Servire portal.**
-   [x] Keep detailed Nuntius diagnostic/control-plane traffic out of the normal
    Servire Operations Log; participating services retain only concise operational
    send/ack/state entries where useful.

**N8.3 validation:** Rogare Trace-state recovery across service reconnect/restart
is complete. With an active Vestigare Trace, Rogare was restarted and recovered
the authoritative recording state; Vestigare retained the active recording and
Rogare exposed only the valid Stop action.

**Exit condition:** **COMPLETE through N8.3.** The routing core, Nuntius-owned
diagnostics UI within Servire, and Trace-state reconnect/recovery behaviour are
implemented and live validated.

## Phase 4 --- Pontis Request/Response Integration

Complete the external control-command path.

-   [x] Recognise client-originated `\obt`.
-   [x] Route it directly to Nuntius.
-   [x] Keep it outside normal model ask/answer processing.
-   [x] Preserve client/session/request correlation.
-   [x] Receive returned query results from Nuntius.
-   [x] Deliver each result to the correct originating client/session.

**Live validation:** `\obt repetere list` was successfully issued from both
Rogare and the external Pi HTTP client. The control result returned over the
client's originating HTTP path and did not enter the model conversation.

**Exit condition:** **COMPLETE for N5.** An external client can issue a
request/query command and receive the authoritative response through the common
Lumen control plane.

## Phase 5 --- Repetere / Moderari Control Migration and Replay Fidelity

Move existing control behaviour onto Nuntius and establish faithful
system-context replay.

-   [x] Migrate existing **Repetere** `\obt` behaviour to the common path.
-   [x] Migrate existing **Moderari** `\obt` behaviour to the common Nuntius path. **N7 implemented and live validated.**
-   [x] Perform the defined Vestigare system-prompt investigation using
    baseline/Default, `Pass-through`, and `Custom`. **Completed during N7+.**
-   [x] Preserve an incoming system prompt as provenance if Moderari supersedes it,
    but exclude the superseded prompt from Replay reconstruction.
-   [x] Ensure Trace unambiguously identifies the effective system prompt that
    reached the model. **Corrected and validated during N7+.**
-   [x] Correct the effective-prompt evidence boundary. **Authoritative effective
    prompt provenance is now correlated to execution and retained separately from
    superseded incoming provenance.**
-   [x] Ensure current `\obt` control messages remain outside conversational Trace.
-   [x] Implement a session-scoped Moderari Pass-through override. **Live validated concurrently: Pi changed to Pass-through while Rogare remained Default.**
-   [x] Require each Replay execution to create a new isolated Replay session.
-   [x] Have Repetere issue the Pass-through override through Nuntius for that new
    Replay session before replay.
-   [x] Require positive Moderari acknowledgement before model interaction begins.
-   [x] Initialise the Replay session only from the source Trace and explicitly
    defined experimental conditions.
-   [x] Replay the effective system prompt captured in the source Trace.
-   [x] Verify sequential Replay runs cannot inherit context or temporary state from
    preceding runs.
-   [x] Verify Replay never substitutes the current Moderari Default or current saved
    Custom prompt.
-   [x] Verify Replay reconstructs exactly one active effective system prompt and
    never combines a superseded incoming prompt with its Moderari replacement.
-   [x] Run at least one deliberately system-prompt-sensitive behavioural validation
    capable of exposing duplicate/incorrect system context.
    **Completed through the recorded Pi/Moderari provenance validation: the incoming Pi
    system prompt was retained as superseded provenance and exactly one Lumen-generated
    effective system prompt was marked as replay input.**

-   [ ] Add authoritative provider and exact model identity to each Vestigare
    `trace_recordings` document.
-   [ ] At actual replay start, require Repetere to read the recorded model, verify it
    against Praebere's cached catalogue, and select/reserve it through the normal
    Pontis/Praebere lifecycle when no global model is selected.
-   [ ] Allow Repetere to proceed when the active global model exactly matches the
    recorded model; reject and visibly log a controlled conflict when it differs.
-   [ ] Have Fiducia apply the same model check at scheduled execution time and record
    a failed run, without changing global model state, when the models differ or the
    recorded model is unavailable/missing.

**Additional completed boundary work:** Vestigare is limited to one active
M0.1 recording at a time and concurrent traffic from other sessions is excluded
from the bound Trace. Rogare/Vestigare recording state is propagated through
Nuntius rather than a new Trace-status polling loop.

**Remaining M0.1 Trace-start work:** Pontis session identity must be treated as
authoritative for recording selection. Vestigare must expose eligible active
sessions, bind the Trace explicitly to the selected Pontis `session_id`, prevent
Trace Start when no eligible session exists, and prevent a new complete-session
Trace from being started after the selected session has already performed its
first model interaction.

**Exit condition:** Replay reproduces exactly the recorded effective system
prompt in a newly created isolated Replay session; superseded incoming system
prompts remain provenance but are not replayed, and no prior Replay state is
inherited. Replay control behaviour must remain consistent with the M0.1 shared
concurrent-session execution-condition rules validated in Phase 10.

## Phase 6 --- Praebere Discovery and Model Selection

Move provider/model control onto the common command path.

-   [x] Complete the N9.1 Praebere current-state audit.
-   [x] Discover Ollama models at startup and only on explicit **Refresh Models**;
    serve routine queries, selection validation and UI polling from Praebere's cached catalogue.
-   [x] Treat the configured model as an optional preferred model and never silently
    select or substitute it for a client.
-   [x] Finalise `\obt providers`, `\obt models` and
    `\obt praebere model select <model>` through the common Nuntius path.
-   [x] Make Praebere authoritative for provider/model operations, runtime-global
    selection, reservation, activation, residency ownership and derived selection state.
-   [x] Return results to the originating session and configured listeners without
    broadcasting them to unrelated services.
-   [x] Distinguish available, preferred, selected, reserved, locked, resident and
    residency-ownership state.
-   [x] Require every session to reserve the authoritative selected model explicitly;
    selecting the same model is idempotent and selecting a different model while any
    open reservation exists is rejected.
-   [x] Treat first ask as model activation, demand-load a non-resident selected model,
    and lock selection while model execution is active.
-   [x] Cancel only the requesting session's reservation when activation fails and
    communicate the controlled load failure and resulting authoritative state.
-   [x] Persist Praebere runtime state in MongoDB and reconcile it with Pontis on
    start/restart; remain fail-safe when authoritative reconciliation is unavailable.
-   [x] Implement Pontis client/admin session lifecycle commands and session-management UI,
    plus Rogare End Session/New Session closure behaviour.
-   [x] Release Praebere execution/reservation state on session closure and unload only
    model residency that Praebere caused after its final retaining session ends.
-   [x] Implement the compact Praebere-owned UI and Servire tab, including cached polling,
    Refresh Models, reconciliation and guarded reset operations.
-   [x] Validate Rogare and external-client query, selection, execution and session-end paths.
-   [x] Validate manual Rogare reconnect by re-entering the Pontis session ID; session
    context survives although Rogare's prior local conversation display is not restored.
-   [x] Close Nuntius N8 diagnostics and live N10 routing/regression validation, including
    bounded history and Pontis-local rejection of external `\obt rogare start`.
-   [x] Confirm the M0.1 tool-responsibility boundary: an external client owns its own tool declaration, execution and result handling.
-   [x] Confirm Rogare has no native tool environment and uses an external tool provider.
-   [x] Validate Pi as the supported M0.1 tool provider for Rogare in its expected/default installation location.
-   [x] Do not claim Rogare support for alternative tool providers in M0.1.

**M0.1 model lifecycle:** provider/model selection remains runtime-global rather than
per-session. Each session must explicitly reserve the same authoritative selected model.
Any open reservation prevents changing to a different model; active model execution adds
the execution lock. Selection becomes available only after the final reserving session ends.

**M0.1 session lifecycle:** Pontis exposes explicit client and operator closure.
The Servire authorization heartbeat remains a separate distribution-security
mechanism and does not determine conversational-session liveness.

**Future development:** per-session provider/model selection remains outside the required M0.1 boundary.

**Exit condition:** **COMPLETE.** Praebere discovers actual Ollama models and owns
authoritative runtime-global model state; Rogare and external clients use the same Nuntius
path; reservation, execution locking, persistence, reconciliation, demand residency,
session release and the compact operational UI have been live validated.

## Phase 7 --- Moderari System-Prompt Policy

Make system-prompt handling an explicit experimental condition.

-   [x] Implement `Pass-through`.
-   [x] Preserve `Moderari Default`.
-   [x] Implement `Custom` as an exact researcher-defined system-prompt
    condition.
-   [x] In `Custom`, remove the incoming client system prompt without
    carrying its tool instructions/descriptions into the selected Custom
    prompt.
-   [x] In `Custom`, do not add, remove, rewrite, merge, normalise or
    otherwise augment the selected Custom prompt.
-   [x] Keep tool availability separate from tool instructions contained in
    the Custom system prompt.
-   [x] Require any desired tool descriptions or tool-use instructions to be
    explicitly present in the researcher-defined Custom prompt.
-   [x] Implement the System Prompt Policy UI as a Moderari-owned
    operational surface.
-   [x] Expose that Moderari UI within Servire's Moderari tab without making
    Servire a control intermediary.
-   [x] Add explicit editor working-copy versus active-runtime-state
    visibility.
-   [x] Require explicit `Apply` before Custom editor contents become
    Moderari's active prompt.
-   [x] Treat `Apply` as a local Moderari configuration action rather than a
    Servire/Nuntius round trip.
-   [x] Verify Pass-through does not alter client system context.
-   [x] Verify Custom reaches the model as the last explicitly Applied
    editor contents, subject only to unavoidable transport
    encoding/serialization; the Applied prompt need not correspond to a
    stored saved prompt.
-   [x] Verify a Custom prompt without tool instructions is not silently
    augmented when tools are available.
-   [x] Verify a Custom prompt containing researcher-supplied tool
    instructions does not acquire a second Moderari-generated tool
    block.
-   [x] Verify the effective system prompt appears in Trace.

**Exit condition:** system-prompt behaviour is explicit, selectable and
reproducible; `Custom` is demonstrably the exact researcher-defined
prompt rather than a Moderari-generated or tool-augmented derivative.

**N7+ status 2026-08-30:** Moderari policy state is session scoped. Default, Pass-through and exact Custom are implemented; saved-prompt working-copy/persistence separation is implemented; effective system-prompt provenance is explicit in Trace; and the session-scoped UI is exposed through Servire. Concurrent Rogare/Pi validation confirmed one session can change policy without altering another established session or the new-session default.

## Phase 8 --- Saved System Prompts

Complete the M0.1 Custom-prompt persistence and reuse workflow.

-   [x] Add a dedicated MongoDB collection for Moderari saved system
    prompts.
-   [x] Define a minimal schema containing stable `prompt_id`,
    human-readable `name`, `content`, `owner_id`, `scope`, `created_at`
    and `updated_at`.
-   [x] Use a simple/default M0.1 owner identity while retaining `owner_id`
    as an explicit future-compatibility field.
-   [x] Retain an explicit `scope` field without implementing
    multi-user/organisational policy in M0.1.
-   [x] List/select saved prompts.
-   [x] Load a selected prompt into the Custom editor as a working copy.
-   [x] Ensure editing the working copy does not mutate the saved object or
    active runtime configuration.
-   [x] Implement `Apply` independently of persistence.
-   [x] Implement `Save As…` to create a new saved prompt without
    overwriting the source prompt.
-   [x] Implement explicit `Update Saved` for replacement of an existing
    saved prompt.
-   [x] Implement explicit `Delete Saved`.
-   [x] Ensure `Save As…`, `Update Saved` and `Delete Saved` do not silently
    alter Moderari's active runtime prompt.
-   [x] Ensure `Apply` does not silently create or update a saved prompt.
-   [x] Retain the actual applied prompt in Trace regardless of later
    saved-prompt changes.
-   [x] Do not store active runtime state as an `is_active` property of a
    saved-prompt document.

Do not add prompt sharing, organisational prompt libraries, mandatory
organisational prompts, permission policy or sophisticated prompt
version history in M0.1. Those are future development concerns.

**Exit condition:** researchers can safely reuse and modify controlled
system-prompt conditions while saved definitions, editor working copies
and Moderari's active runtime configuration remain explicit and
independent.

## Phase 9 --- Experiment Structure

Implement the Experiment relationship required for controlled repeated
execution.

-   [ ] Introduce Experiment as an explicit concept.
-   [ ] Associate an Experiment with its source Trace.
-   [ ] Associate replay-created Traces with Experiment runs.
-   [ ] Have Fiducia coordinate repeated runs against an Experiment.
-   [ ] Harden Fiducia startup against stale PID files: if the PID file
    exists, verify that the recorded PID belongs to a live Fiducia
    process; if not, treat the file as stale, remove/replace it, log the
    recovery, and continue startup. A genuinely running Fiducia instance
    must still prevent duplicate startup. **Current live behaviour: stale PID
    recovery is now failing on every managed start; likely lifecycle/shutdown-order
    race and must be investigated in the next Fiducia/Servire pass.**
-   [ ] Correct Fiducia managed log cleanup. Servire currently invokes
    `clear-logs`, but Fiducia's CLI rejects that argument and exits with code 2.
    Investigate this alongside the PID/lifecycle boundary rather than as an
    unrelated feature.
-   [ ] Ensure Fiducia PID/runtime files are container-runtime state and are
    not persisted as durable application data in the Docker distribution.
-   [ ] Ensure every Experiment run receives its own Replay session rather
    than reusing the session of another run.
-   [ ] Surface run state as `MATCHED`, `DIVERGED`, or
    `FAILED / INCOMPLETE`.
-   [ ] Define an explicit recovery lifecycle for a failed staged Replay. A failed
    run must remain preserved as evidence while the staged experiment can be
    retried, reset/recovered, or unstaged without becoming indefinitely stuck in
    an ambiguous `FAILED - COMPLETED` state.
-   [ ] Ensure retrying a failed staged Replay creates a fresh isolated Replay
    session/run and does not overwrite, discard, or silently mutate the preserved
    failed-run evidence.
-   [ ] Make the Repetere UI distinguish clearly between the terminal state of an
    individual failed run and the current actionable state of its staged Experiment.
-   [ ] Surface first divergence point where available.
-   [ ] Present Experiment/run relationships clearly in the UI.
-   [ ] Preserve the distinction between Repetere divergence detection and
    future Aestimare assessment.

**Exit condition:** a researcher can create repeated controlled
executions and see the resulting evidence as one coherent Experiment.

## Phase 10 --- Multi-Session Isolation Validation

Investigate concurrent-session transaction isolation under the M0.1 shared
execution-condition model.

-   [x] Run at least two simultaneous active sessions. **Rogare and Pi were active concurrently.**
-   [x] Verify transactions initiated within each session remain bounded to their
    originating client/session for the tested ordinary ask/answer paths.
-   [x] Verify session-specific conversational responses and Trace evidence do not
    cross between the tested Rogare and Pi sessions. **An earlier global-recording
    defect was found, then corrected by binding Vestigare to one session.**
-   [x] Verify Pontis/Nuntius request-response correlation remains bound to the
    originating session for the tested `\obt repetere list` paths.
-   [x] Validate independent Moderari system-prompt policy state across concurrent sessions. **Rogare remained Default while Pi was changed to Pass-through.**
-   [x] Verify changing one established session's Moderari policy does not change another established session or the default for new sessions.
-   [x] Establish the M0.1 concurrent provider/model selection constraint independently:
    provider and selected-model state are runtime-global. Per-session provider/model
    isolation is not an M0.1 requirement and must not be inferred from Moderari's
    session-scoped policy behaviour.
-   [ ] Make Pontis `session_id` the authoritative Vestigare Trace-binding identity.
-   [ ] Have Vestigare obtain and display eligible active Pontis sessions for Trace Start.
-   [ ] Disable/reject Trace Start when there are no eligible active sessions.
-   [ ] When exactly one eligible session exists, bind/display that session explicitly.
-   [ ] When multiple eligible sessions exist, require the researcher to select the
    session to record; never infer selection from recency, creation order or activity.
-   [ ] Persist the selected Pontis `session_id` as the Trace session binding.
-   [ ] Define eligibility for a complete-session Trace as an established session that
    has not yet performed its first model interaction.
-   [ ] Reject complete-session Trace Start after the selected session has already
    performed model interaction rather than silently creating an incomplete Trace.
-   [ ] Validate the normal sequence: client connects → Pontis assigns `session_id` →
    researcher starts Trace → Vestigare binds Trace → first model interaction.
-   [ ] With multiple sessions active, verify only traffic matching the selected
    `session_id` enters the active Trace.

**M0.1 Trace limitation:** Vestigare supports one active Trace recording at a
time. Session filtering/ownership is implemented, but the complete M0.1 recording
lifecycle is not yet complete: Trace Start must be driven by explicit eligible
Pontis-session selection and must occur before the selected session's first model
interaction. Other sessions continue normally but are not recorded. Concurrent
independent Trace recordings remain future development.

Any demonstrated session leakage is an M0.1 blocker.

**Exit condition:** Moderari session-policy isolation and transaction/client/Trace
isolation have live evidence; Vestigare Trace Start is explicitly bound to an eligible
Pontis session before its first model interaction; zero-, one- and multi-session start
behaviour is validated; and Praebere provider/model state remains explicitly runtime-global for M0.1,
with model selection locked while any active execution session exists.

## Phase 11 --- Runtime Authorization and Distribution Security

Implement the external-distribution trust boundary defined by **M0.1
Runtime Authorization and Code Protection**.

-   [ ] Define Distribution ID and Service Group UUID handling.
-   [ ] Provision an installation-specific cryptographic identity.
-   [ ] Register the installation public identity with Illuminates.One.
-   [ ] Implement Servire → Illuminates.One authorization over TLS/HTTPS.
-   [ ] Authenticate the Servire request using the installation identity.
-   [ ] Implement Illuminates.One signed authorization responses and Servire
    verification.
-   [ ] Implement nonce/challenge freshness protection.
-   [ ] Implement authorization lease, renewal, grace and expiry states.
-   [ ] Verify deliberate heartbeat suppression cannot provide indefinite
    normal operation.
-   [ ] Define and implement short-lived Servire-authorised
    protected-service startup.
-   [ ] Ensure protected services reject unsupported/unauthorised startup as
    an authorised external runtime.
-   [ ] Encrypt protected operational configuration at rest.
-   [ ] Couple normal configuration unlock/use to valid runtime
    authorization without embedding a permanent master unlock secret in
    the distribution.
-   [ ] Prefer in-memory use of decrypted protected configuration.
-   [ ] Validate that copying release/install material does not
    automatically reproduce an authorised installation.
-   [ ] Inspect final distributed artifacts for private Illuminates.One
    authority, master secrets or hard-coded bypass material.
-   [ ] Document the explicit non-goal of absolute tamper resistance on
    researcher-controlled hardware.

**Exit condition:** a provisioned installation can establish signed
runtime authorization, operate under a bounded lease, start protected
services through Servire, use protected configuration only through the
authorised runtime path, and fail closed after authorization expiry;
copied distribution material alone is insufficient to create another
authorised installation.

## Phase 12 --- Internal M0.1 Acceptance Validation

This phase is Lumen's **internal acceptance testing** for the M0.1
release candidate. It is performed before the distribution is supplied
for external research use and is not intended to replace or predetermine
external researcher evaluation.

Run the complete M0.1 research workflow as an internal acceptance
exercise.

A representative validation should demonstrate:

1.  client/Rogare connection;
2.  provider/model discovery through Nuntius;
3.  model selection;
4.  selectable system-prompt policy through the Moderari-owned UI,
    including when exposed in Servire;
5.  saved Custom prompt selection and loading into the editor as a
    working copy;
6.  verification that editing the working copy does not alter the saved
    prompt or active Moderari configuration;
7.  explicit Apply of the current editor contents;
8.  verification that Save As creates a new prompt while Update Saved is
    the only explicit replacement path;
9.  verification that saved-prompt persistence and active runtime state
    remain independent;
10. verification that Custom reaches the model without Moderari tool
    extraction, merging or augmentation;
11. verification that tool availability does not silently alter a Custom
    system prompt;
12. Vestigare Trace Start is unavailable when no eligible active session exists;
13. with one eligible session, Vestigare explicitly displays/binds its Pontis `session_id`;
20. with multiple eligible sessions, Vestigare requires explicit researcher selection
    and never guesses from recency/activity;
21. Trace Start succeeds after Pontis session establishment but before that session's
    first model interaction;
22. Trace Start is rejected for a session that has already performed model interaction;
23. with multiple sessions active, only traffic for the selected Pontis `session_id`
    enters the active Trace;
24. normal model execution and complete-session Trace capture;
25. source Trace selection;
14. session-scoped Replay Pass-through;
15. faithful replay of the source effective system prompt;
16. Experiment creation;
17. Fiducia-coordinated repeated executions;
18. Repetere divergence reporting;
19. clean Trace evidence without internal `\obt` chatter;
26. query responses delivered only to the correct originator/configured
    consumers;
27. simultaneous-session isolation, including confirmation that Moderari system-prompt policy is independently session scoped, changing one session does not alter another session or the new-session default, and remaining provider/model concurrency rules are explicitly validated;
28. a fresh isolated session for every Replay run;
29. no context/state inheritance between repeated Replay runs;
30. incoming versus effective system-prompt provenance is
    distinguishable;
31. exactly one effective system prompt is reconstructed during Replay;
32. a system-prompt-sensitive test confirms replaced and duplicate
    prompts cannot silently contaminate Replay;
33. installation cryptographic identity and signed Illuminates.One
    authorization;
34. nonce/freshness rejection of replayed authorization responses;
35. authorization lease renewal, grace and expiry behaviour;
36. protected-service startup through Servire authorization;
37. protected configuration availability only through the authorised
    runtime path;
38. copied installation material failing to establish another authorised
    installation;
39. absence of Illuminates.One private signing authority, permanent
    master unlock secrets and hard-coded authorization bypasses from
    distributed artifacts;
40. external-client tool declaration/execution remains client-owned,
    while Rogare tool use is validated through Pi in its
    expected/default installation location;
41. Moderari context compaction is exercised across the approximately
    63% utilisation threshold and the effective checkpoint/compacted
    context remains observable in Vestigare;
42. Nuntius diagnostics are available through the Nuntius UI within
    Servire and do not enter the normal Servire Operations Log;
43. remaining Rogare **Moderari heartbeat/progress** polling is either
    documented as an M0.1 Operations Log limitation or, if time permits, moved
    onto Nuntius; Trace ownership/state polling has already been removed;
44. the complete Dockerised Lumen distribution operates successfully on
    a single host machine;
45. Vestigare provider/model provenance and Repetere provider/model
    Replay-binding behaviour have been established by inspection and
    controlled cross-model/provider Replay testing, with no unsupported
    binding or substitution guarantee left implicit.
46. a Git-based release feedback and issue-reporting route is available
    and documented for the external researcher, providing a persistent
    place for comments, bug reports and feature requests rather than
    relying on LinkedIn or other informal messaging.
47. Fiducia startup recovers automatically from a stale PID file whose
    recorded process no longer exists, while still refusing a genuine
    duplicate running instance; Docker packaging does not persist the
    PID file as durable application state.
48. Pontis UI and `\obt` controls can inspect/end sessions and authoritatively
    clear an operator-confirmed orphan without affecting unrelated sessions;
49. Rogare End Session, New Session, stop and restart execute the agreed Pontis
    closure lifecycle;
50. ending one activated session releases only its Praebere registration, and
    model selection unlocks only after the final active execution session ends;
51. Praebere restart restores selected model and active execution registrations,
    remains fail-safe through Pontis reconciliation, and exposes unresolved
    release/reconciliation state;
52. safe Praebere execution reset is rejected while Pontis reports active
    execution sessions and preserves provider/model state when validly used.

**Pre-acceptance evidence already obtained:** client/Rogare and Pi operation,
normal Trace capture, Repetere list control routing, clean `\obt` separation,
Trace session ownership, and multi-client control correlation have all been
exercised during development. They remain part of the formal Phase 12 pass and
are not considered release-candidate acceptance merely because development
smoke tests succeeded.

Regression tests for existing Lumen behaviour should remain green.

**Exit condition:** the M0.1 release candidate passes Lumen's internal
acceptance validation, the workflow is coherent end-to-end, its known
limitations are documented, and an appropriate Git-based channel is
available for external research comments, bug reports and feature
requests.

## Current Known Bugs and Optional M0.1 Polish --- reconciled 2026-09-06

### Bugs / required follow-up

- **Resolved during N8/N9:** Pontis session termination/Praebere release,
  Vestigare session-bound Trace start, Nuntius Trace-status routing, Fiducia stale
  PID recovery and Fiducia `clear-logs` compatibility are closed.
- **Vestigare/Repetere/Fiducia model fidelity:** add authoritative model metadata to
  `trace_recordings` and enforce exact recorded/global model compatibility at actual
  replay start, as specified in Phase 5.
- **Repetere failed staged-Replay lifecycle:** a failed run can remain presented as
  `FAILED - COMPLETED`. Preserve the failed run as evidence, but provide an explicit
  staged-Experiment recovery lifecycle for retry/reset/unstage. Any retry must create
  a fresh isolated Replay session/run and must not overwrite the failed-run evidence.

### If time --- polish, not release blockers

- Move Rogare's remaining Moderari heartbeat/progress status traffic from the
  historical Pontis polling/backchannel path onto Nuntius/`\obt`, reducing
  repetitive Servire Operational Log noise.
- After Praebere provider/model discovery is on Nuntius, have Pontis request and
  present provider/model choices automatically when an external client connects.
- If N9 lands early enough and the implementation remains bounded, consider
  **per-session model selection** as optional M0.1 polish. This is explicitly not a
  release blocker; the required M0.1 behaviour is one runtime-global provider/model
  selection shared by all sessions.
- Continue standardising service configuration filenames/locations to root-level
  `config.yaml` when each service is next modified; do not create standalone
  churn solely for this housekeeping.

## After M0.1 --- Before General Research Release

The following work is not required to define M0.1 but is required or
expected before wider external research distribution:

-   Servire single-active-operator enforcement using a renewable
    operator lease;
-   graceful refusal of a second simultaneous Servire operator;
-   explicit release/exit of the active operator session;
-   recovery from abandoned/crashed operator sessions;
-   final packaging/distribution hardening and documentation, including
    provisioning/revocation operating procedures where required;
-   expose the Servire operator-facing interface over HTTPS/TLS rather than
    plain HTTP as a supported deployment boundary. TLS termination may be
    provided by the deployment/runtime layer; this does not require Servire
    itself to own certificate issuance or management. Internal Lumen
    service-to-service traffic remains a separate transport/security concern.

## Explicitly Deferred Beyond M0.1

-   explicitly labelled partial/mid-session Trace recording and historical Trace
    backfilling for model interactions that occurred before Vestigare recording began;
-   multiple simultaneous Vestigare Trace recordings;
-   Rogare human correctness/quality ratings;
-   Aestimare;
-   multiple Pontis tool providers and unified tool catalogue;
-   bounded-computational-resource behavioural experiments;
-   Servire Operations Log user/internal classification and default
    filtering. Trace-state polling has already been removed in favour of
    Nuntius events; remaining heartbeat/progress polling may be migrated as
    optional M0.1 polish or otherwise remains part of this deferred cleanup;
-   commercial multi-user/operator entitlement;
-   saved-prompt ownership and organisational policy, including
    user-private, organisation-shared, installation-wide and mandatory
    organisational prompt scopes and associated permissions;
-   sophisticated prompt libraries/versioning;
-   durable enterprise messaging for Nuntius;
-   distributed multi-host deployment, including the required service
    discovery, transport, operational control, security, authorization
    and logging validation.
