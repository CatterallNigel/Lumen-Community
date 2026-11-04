# Lumen — Repetere Increment 01: Source Audit and Proposed Contracts

**Date:** 31 October 2026  
**Status:** Updated cross-service audit incorporating verified Vestigare SEP-before-recording behaviour and agreed opaque-SEP client contract; remaining implementation details identified. No source changes made.  
**Sources:** Repetere v0.20.20, Moderari 30 October 2026, Vestigare 18 October 2026, Praebere ExPro Handover Corrected (previously reviewed), Nuntius v0.4.5 (31 October audit), consolidated RPR register, agreed architectural-alignment plan, and subsequent approved design decisions.  
**Code changes:** None.

## 1. Governing invariants

1. Original Setup uses Vestigare's authoritative recorded SEP, not a saved Execution Profile or saved prompt/filter-profile record. Repetere treats the SEP as an opaque artefact: it retrieves and submits it, without interpreting or validating its execution semantics. Moderari/SIE own all SEP validation and activation.
2. Exact Adapter/Filter identity and version, resolved configuration, prompt content, response mode and execution semantics must be preserved. A missing or incompatible executable component is an explicit Moderari failure reported to Repetere, without substitution.
3. New Setup follows the existing client session path: Pontis session, Praebere model reservation/EMD, Moderari applicable Execution Profiles, profile selection, First Ask SEP finalisation and Vestigare acknowledgement.
4. Every replay has a distinct Pontis-owned session ID, independent immutable SEP, Run and Vestigare child trace; original artefacts are unchanged.
5. Repetere owns replay/experiment outcomes and first divergence; Pontis owns authoritative session termination; Moderari owns SEP construction/execution; Vestigare owns SEP/trace persistence.
6. The goal is configuration fidelity, not identical model output. Supported source recordings are assumed to have a valid SEP; no pre-SEP migration is required.

## 2. Source findings

### Repetere

- `src/lumen_repetere/services/replay_runtime.py`: `ReplayPrerequisite.establish_moderari_pass_through()` and `_establish_replay_prerequisite()` impose a universal pass-through prerequisite. This is incompatible with replay of an ADAPTERS_ENABLED SEP. Replace the **preparation boundary**, not the replay/matching engine.
- The same runtime defines `ReplaySessionPreparationLifecycle.prepare_execution_session(required_model)`, `ReplayModelLifecycle.prepare(...)`, and `start()` with `requested_model` override. Current preparation is model-oriented, not SEP-oriented; New Setup needs explicit provider/model and Execution Profile selection, and Original Setup needs an SEP-based route.
- `start()` loads recorded chat exchanges, requires at least one replayable tool call, and allocates a Run and execution session. This is a current behavioural restriction; do not silently remove it during alignment. Test its implications for RPR-01 and future general replay support.
- Existing `TraceControlClient` in `infrastructure/trace_control.py` starts child recordings with `owner=repetere`, source trace, Experiment, Run and session identifiers; captures private exchanges through `POST /trace/recordings/current/exchanges`; and finalises/deletes recordings. Retain this integration and confirm session-to-SEP linkage.
- `ReplayRuntime` maintains explicit cleanup, cancellation, interruption recovery and first-fork paths. Preserve them and extend failure-stage handling rather than rewriting them.

### Moderari

- `src/lumen_moderari/emd_handover.py`: `select_execution_profile()` resolves a current ExPro and its linked Filter Profile, configuring a resolved filter chain. `prepare_first_ask()` requires `_selected_profiles`, checks current ExPro existence, resolves/finalises a plan and registers it. **There is no observed Original Setup branch that reconstructs solely from a recorded SEP.**
- `src/lumen_moderari/sie/session.py`: `SessionExecutionPlan` is a frozen dataclass containing `session_id`, `execution_mode`, `response_mode`, CIM/protocol/model fields, Adapter references (identity/version/CIM version), and ordered Filter references (identity/version/configuration). For enabled execution, the constructor requires complete Adapter/protocol/model fields; disabled execution rejects Adapter and Filter details.
- `SIESessionPlanner.materialise_execution()` currently resolves Filters by identity from the catalogue against `state.finalised_plan`, without an observed explicit equality check against the recorded Filter version at this call site. The reconstruction path must explicitly validate exact version, including Adapters and CIM compatibility, before materialisation.
- `src/lumen_moderari/sie_integration.py`: `register_plan_and_materialise()` requires `plan.session_id == session_id`, enforces SIE state, registers the SEP with Vestigare and transitions to READY only after acceptance. Reuse this gate; a replay-specific session ID requires a new plan with execution-equivalent fields and explicit source provenance, not a verbatim original session ID.
- Existing SEP schema does not include an originating Execution Profile identity. This is consistent with the agreed reconstruction model.

### Praebere (previous source review)

- Existing session-scoped model reservation and EMD handover to Moderari are the intended Original Setup path. Praebere relays applicable Execution Profiles; Repetere may ignore the returned list for Original Setup. Model activation remains on the ordinary First Ask lifecycle.
- **Expected change:** none unless integration testing identifies a defect. This statement reflects the earlier Praebere review; the Praebere archive was not re-opened for this Nuntius-focused revision.

### Nuntius v0.4.5 (verified from attached source)

- `models/control.py`: `ControlEnvelope` requires `request_id`, `timestamp`, `origin`, `command`; `session_id` is nullable. The destination is the token immediately after `\obt`, with at least three tokens required. The unchanged envelope is forwarded.
- `api/control.py`: `POST /control/obt` resolves the service through the Servire-supplied control catalogue; duplicates of `request_id` are rejected (409); an unknown destination yields 404. It forwards to the registered owner and returns the owner HTTP status/body with `x-lumen-control-request-id`.
- `services/control.py`: HTTP errors from the destination remain authoritative; network/transport errors become 504 with `outcome_confirmed=False`. A 504 therefore **does not prove the registration failed**. Repetere must reconcile uncertain outcomes using idempotent registration and a retry/status strategy.
- `api/control.py` and `config.py`: ordinary control timeout defaults to 5 seconds; selected model lifecycle operations use 300 seconds. The proposed lightweight recorded-SEP registration is not in the existing extended-timeout cases. Avoid model loading or First Ask work inside that command.
- Nuntius is a generic router; it does not parse SEP semantics. **No Nuntius code changes are currently required**, subject to tests.

### Vestigare

- `src/lumen_vestigare/session_plans.py`: `SessionPlanRegistry.register()` persists a canonical deep copy of the plan keyed by unique session ID, rejects conflicting plans, and checks required enabled-plan fields. `attach_to_trace()` embeds the authoritative plan in a recording.
- `src/lumen_vestigare/session_plan.py`: `store_plan()` writes `session_execution_plan` and `execution_plan_sha256` to a recording and rejects conflicting hashes; `accept_plan()` registers a plan and only returns success for an active recording with matching session ID.
- `src/lumen_vestigare/api/control.py`: Nuntius-addressable `session plan` command accepts Moderari's SEP. The **write/acknowledgement** path is present. A dedicated supported read API returning the complete SEP by source recording ID has **not yet been verified** and must be established, without relying on Moderari's diagnostic export.
- **Verified SEP-before-recording behaviour:** `accept_plan()` first registers the SEP in the session-plan registry, then returns `recorded: false` if no matching recording is active. It does **not** reject that registration. `RecordingManager.start()` obtains the plan for the session and calls `_attach_session_plan()` to copy it into the new recording. The existing `tests/test_session_plan.py` asserts a successful control response with `accepted: true, recorded: false`. Consequently, recording need not precede SEP registration. `accepted` and `recorded` are distinct statuses; the Moderari acknowledgement contract must use the actual accepted result, not require `recorded: true`.

## 3. Proposed contract boundaries (design; not existing APIs)

### 3.1 Vestigare → Repetere: authoritative source package

`get_replay_source(recording_id)` returns source recording ID, original session ID, client-boundary exchanges, complete recorded SEP, recorded SHA-256 and source trace metadata. Validate integrity and identity; do not reconstruct from transformed Moderari evidence. Decide whether to extend an existing recording read endpoint or add a dedicated retrieval route after reviewing all existing Vestigare APIs.

### 3.2 Repetere → Moderari: recorded-SEP registration (AGREED)

**No separate general-purpose preflight API.** Repetere is an automated client, and authoritative component validation at First Ask is the same gate used for all clients.

Sequence: Pontis creates a fresh session; Repetere retrieves the immutable source SEP and its integrity metadata from Vestigare without interpreting the SEP; Repetere reserves the SEP's recorded provider/model through Praebere; Praebere supplies authoritative EMD to Moderari and returns available Execution Profiles, which Repetere silently ignores for Original Setup; Repetere submits the recorded execution definition to Moderari via Nuntius for association with the new session.

Proposed command (final spelling to align with Moderari's parser):

```text
\obt moderari session execution-plan register <JSON payload>
```

Nuntius requires a standard control envelope containing `request_id`, `timestamp`, `session_id`, `origin`, and `command`. Because its envelope has no separate payload field, the SEP payload must be encoded in the `command` string (as Praebere does with its EMD handover), unless a separately approved envelope change is made. Avoid assuming arbitrary out-of-band JSON is currently supported. The `session_id` in the envelope and payload must agree.

Moderari verifies that the new session exists, has the authoritative Praebere provider/model reservation, and is not already bound to a conflicting recorded definition or immutable SEP. Repetere performs no Adapter, Filter, mode or protocol validation; these checks belong exclusively to Moderari/SIE. It associates an execution-equivalent **copy** of the recorded SEP with the new session ID, without an Execution Profile lookup. The original SEP and SHA-256 remain untouched.

Proposed positive acknowledgement (logical fields, not existing implementation):

```json
{
  "session_id": "<new-session-id>",
  "status": "accepted",
  "execution_plan_source": "recorded_sep",
  "materialised": false,
  "session_ready": false
}
```

`accepted` means the recorded definition is held for this session; **it does not mean that SIE has materialised an SEP or that the session is READY**. Repeated identical registrations should be idempotent; conflicting registrations must be rejected. A registration must never overwrite an already materialised SEP, change the provider/model reservation, or bypass Vestigare acknowledgement.

**Transport uncertainty:** Nuntius forwards the owner's status/body, but on a network error returns 504 with unconfirmed outcome. Repetere must not interpret that as a definite rejection. Use an idempotent command and a defined status/reconciliation path; each retry has a fresh Nuntius `request_id`, while the target session and definition identity remain stable.

Suggested failure categories (proposals, not implemented): `SOURCE_SEP_INVALID`, `SOURCE_SEP_INTEGRITY_FAILURE`, `SESSION_NOT_FOUND`, `SESSION_MODEL_NOT_RESERVED`, `MODEL_RESERVATION_MISMATCH`, `RECORDED_SEP_CONFLICT`, `ADAPTER_NOT_FOUND`, `ADAPTER_VERSION_UNAVAILABLE`, `FILTER_NOT_FOUND`, `FILTER_VERSION_UNAVAILABLE`, `CIM_VERSION_UNSUPPORTED`, `PROTOCOL_INCOMPATIBLE`, `MODEL_UNAVAILABLE`, `EXECUTION_CONFIGURATION_UNSUPPORTED`, `VESTIGARE_REGISTRATION_FAILED`. Include component identity/version and stage where applicable.

### 3.3 Original Setup: authoritative First Ask activation (AGREED)

On First Ask, Moderari/SIE uses the accepted recorded definition **instead of resolving an Execution Profile**. It authoritatively validates the exact Adapter/Filter identities and versions, CIM/protocol compatibility, and execution configuration, with no substitution. The replay SEP has the new Pontis session ID and execution-equivalent fields. Preserve source recording/session/SEP hash as linked provenance, without modifying the original SEP.

On Repetere First Ask, recording is started and Moderari/SIE validates and materialises the immutable replay SEP, sending it to Vestigare through the existing acknowledgement gate. Vestigare also supports SEP registration before recording starts, with later attachment by session ID; there is no architectural requirement that recording exist before SEP registration. Vestigare acknowledgement remains mandatory before SIE transitions to READY and execution proceeds. A failed First Ask validation is a structured Moderari failure reported to Repetere, with New Setup still available. This is the normal First Ask validation rule, not a special replay-only policy.

### 3.4 New Setup: normal client configuration

Use the same Pontis/Praebere/Moderari operations as Rogare: session creation, model reservation and EMD, applicable ExPro discovery, selected ExPro delivery, then first recorded message triggers SEP finalisation and Vestigare acknowledgement. Repetere must not duplicate ExPro applicability logic.

### 3.5 Repetere preparation state

Proposed explicit states: `SOURCE_LOADED`, `SESSION_CREATED`, `MODEL_RESERVED`, `CONFIGURATION_SELECTED` (New Setup) / `RECORDED_CONFIGURATION_ACCEPTED` (Original Setup), `FIRST_ASK_PENDING`, `CHILD_RECORDING_STARTED`, `SEP_ACKNOWLEDGED_READY`, `REPLAYING`, terminal success/failure/cancelled. Recording begins on Repetere First Ask; registration and attachment are separate Vestigare operations. Separate failure-attempt persistence from successful replay evidence. **Whether failed registration or First Ask creates a Run or separate preparation-attempt record remains deliberately undecided.**

## 4. Critical integration decisions to settle before coding

1. **SEP read:** exact Vestigare API and returned integrity/provenance envelope.
2. **Reconstruction command:** Moderari Nuntius `\obt` parser, accepted-definition session state, idempotency and authoritative First Ask activation; no separate preflight API.
3. **Recording/SEP association (verified):** Vestigare accepts an SEP before recording exists (`accepted: true`, `recorded: false`) and attaches it at recording start by session ID. Repetere starts recording on First Ask; confirm precise call ordering in the integration tests, without introducing a prerequisite that contradicts Vestigare.
4. **Version enforcement:** exact Filter/Adapter version, CIM version, protocol representation and config validation, with no current-profile lookup.
5. **Disabled SEP (test coverage, not a Repetere decision):** Repetere treats `ADAPTERS_DISABLED` and `ADAPTERS_ENABLED` SEPs identically as opaque inputs. Moderari/SIE enforce the schema and implement the recorded configuration without adding or substituting Adapters or Filters. Verify response-mode fidelity where present in the recorded SEP.
6. **Tools:** source SEP tool definitions/configuration versus actual available tool host at replay time; classify missing tools explicitly.
7. **Failure evidence:** whether a failed registration or First Ask preparation has a Run, separate attempt record or both; avoid a misleading completed child trace.
8. **Provenance (agreed):** retain original recording ID, original session ID and source SEP SHA-256 in Run/child-trace metadata; new replay session and SEP have their own identities/hash. Moderari/SIE compare resolved execution fields, excluding new session/recording/Run identifiers and resulting SEP hash. Repetere does not perform semantic SEP comparison.
9. **Session cleanup:** all partial-setup failure paths close the Pontis session and release Praebere reservation through existing authorities.
10. **Tool replay and divergence (agreed; later integration check):** on a matching model tool call, Repetere returns the recorded Vestigare tool response without executing the tool. On a different call, Repetere records first divergence and switches to live proxy pass-through. The move of Pi SDK tool hosting to Rogare may affect post-divergence dispatch; verify in Increment 04, not as an Increment 01 prerequisite. The current engine's requirement for at least one recorded tool call is an observed implementation limitation to preserve unless separately scoped for change.

## 5. Increment 01 proposed acceptance tests

- Original Setup succeeds with deleted originating ExPro, Filter Profile and saved prompt definition, provided exact executable component versions remain.
- Missing exact Adapter/Filter version fails with a stable diagnostic and no substitution; New Setup remains selectable.
- Reconstructed SEP has a new session ID but identical execution-equivalent fields, including literal System Prompt configuration and response mode.
- Original Setup never selects or looks up the current ExPro; it may receive and ignore Praebere's ExPro list. New Setup uses the existing client selection route.
- On First Ask Repetere starts the child recording, and Moderari/SIE validate and materialise the SEP. Execution is blocked until Vestigare acknowledges the new SEP and SIE is READY; acceptance of recorded configuration alone is not READY. Separately test SEP registration before recording start and later session-ID attachment.
- Vestigare persists independent original/replay plans and child trace linkage without modifying original evidence.
- Both ADAPTERS_ENABLED and ADAPTERS_DISABLED configurations are exercised; Repetere does not branch on SEP internals. Malformed/injected SEP, session/model mismatch, missing exact executable version and incompatible execution definition are rejected by Moderari/SIE with structured errors and no substitution.
- Partial preparation failure and cancellation use Pontis-owned session termination and Praebere lifecycle, without leaking reservations.
- Existing Experiment/Run/fork, trace capture, cleanup, Fiducia-facing interfaces and tests remain operational.

## 6. Implementation sequencing after contract approval

1. Define source SEP retrieval/integrity contract (Vestigare).
2. Implement/test Nuntius-routed recorded-SEP registration and First Ask activation with exact version enforcement (Moderari).
3. Define replay preparation protocol and structured errors (Repetere), preserving legacy execution functionality until replacement is integrated.
4. Verify First Ask recording initiation, SEP acknowledgement, and session-ID association against the already supported Vestigare SEP-before-recording path.
5. Add contract tests and failure-path fixtures before changing the runtime execution path.

## 7. Verified Nuntius transport contract and integration tests

The wire request to `POST /control/obt` has this shape (example only):

```json
{
  "request_id": "<unique-request-id>",
  "timestamp": "<ISO-8601-timestamp>",
  "session_id": "<new-session-id>",
  "origin": "repetere",
  "command": "\\obt moderari session execution-plan register {\"session_id\":\"<new-session-id>\",\"recorded_sep\":{...}}"
}
```

The `command` field is a **string** containing the command and serialised JSON; the example uses an abbreviated `{...}` and is not a complete runnable request. Final command grammar, payload size, escaping and Moderari parser behaviour must be tested against existing service conventions. Avoid logging sensitive prompt content inadvertently: Nuntius v0.4.5 logs complete outbound and inbound envelopes, including command contents. This is an operational/security consideration for transporting a complete SEP, which can contain system prompt text. A reference-based transfer or logging redaction may be preferable if supported; that decision remains open.

Add tests for routing and response transparency; owner 4xx/5xx propagation; 504 uncertain outcome; idempotent replay of a command with a new request ID; conflicting SEP rejection; mismatch of envelope/payload session ID; short-timeout behaviour; no Nuntius SEP interpretation; and prevention of full prompt disclosure in logs if the payload is transmitted inline.

## 8. Decision status and remaining gates

**Approved:** Repetere is an ordinary automated client; Praebere reservation/EMD path reused; ExPro list ignored for Original Setup; recorded SEP held against new session by Moderari; positive acknowledgement is accepted/not materialised/not READY; First Ask is the authoritative SIE validation and Vestigare acknowledgement gate; no overwrite, reservation mutation or conflicting second definition; Nuntius `\obt` internal messaging.

**Verified:** Vestigare registers an SEP even with no active recording, returns `recorded: false`, and attaches a pre-existing session SEP when recording starts. Repetere treats SEP internals as opaque. Matching tool calls receive recorded responses; divergent calls enter live pass-through, with Rogare tool-host routing deferred to Increment 04.

**Still to verify/decide:** exact Vestigare SEP retrieval route; final Moderari command spelling and JSON grammar; prompt-bearing payload transport/logging policy; uncertain-outcome reconciliation; precise source provenance persistence; failure evidence/Run creation policy; tool-host dependencies; complete end-to-end integration tests.

**Exit gate:** settle these remaining interfaces and policies, then implement and test Increment 01, returning synchronised complete packages for affected services. No source code has been changed in this audit revision.

## 9. Clarifications approved after source review (31 October 2026)

- **SEP is opaque to Repetere.** It is not Repetere's responsibility to determine whether a saved SEP is enabled/disabled, whether Adapters are absent, or whether Filters are available. Repetere submits the recorded execution definition and relays Moderari's outcome.
- **Historical validity is not a security bypass.** A SEP originally generated by SIE is authoritative historical evidence. The new-session payload still passes Moderari/SIE's schema, session/model binding, exact component/version and compatibility checks, including for malformed or injected `\obt` payloads.
- **First Ask is the common authoritative gate.** Repetere follows the same validation/materialisation and Vestigare acknowledgement requirements as any client. No separate preflight API is introduced.
- **Recording ordering is no longer an open architectural decision.** Source inspection verifies SEP registration before recording and attachment on start; test the agreed First Ask path without imposing an incorrect prior-recording requirement.
- **Tool semantics remain unchanged.** Expected calls are answered from recorded trace results. Divergence transitions to live tool pass-through, whose compatibility with Rogare-hosted tools is a deferred Increment 04 investigation.

**Verification references:** Vestigare `src/lumen_vestigare/session_plan.py::accept_plan`, `src/lumen_vestigare/session_plans.py::SessionPlanRegistry.register/attach_to_trace`, `src/lumen_vestigare/recording.py::RecordingManager.start`, and `tests/test_session_plan.py` (accepted true / recorded false).
