# Lumen — Repetere Architectural Alignment Implementation Plan

**Document status:** Proposed implementation plan for review; architectural principles agreed, detailed interfaces subject to verification.  
**Baseline:** 31 October 2026  
**Branch:** `research-foundation-next`  
**Release target:** `NEXT` (placeholder)  
**Primary package:** Repetere v0.20.20  
**Supporting packages:** Moderari 2026-10-30; Vestigare 2026-10-18  
**Governing requirements:** RPR-01–RPR-09, with dependencies on MOD-08–10, MOD-18, PTR-01/03/06/08, PBR-16/18/19/21, VTR-01–06 and later FID-01–02.

## 1. Purpose and scope

Align Repetere with the operational Moderari Session Interaction Engine (SIE), immutable Session Execution Plan (SEP), and current Pontis–Praebere–Moderari–Vestigare lifecycle. Preserve existing Experiment → Run → child-Trace organisation, replay controls, matched/divergent behaviour, recording, cleanup, and Fiducia-facing compatibility wherever possible.

This plan includes required changes in associated services, contract verification, test strategy, delivery sequence, failure handling and acceptance gates. **No code modification is authorised by this document.** Implementation begins only after review and approval of the relevant increment.

### Out of scope

- Backward compatibility for recordings lacking an SEP: **every supported Vestigare recording has a valid SEP at recording time**.
- Guaranteeing identical model outputs on replay. The objective is **configuration fidelity**, with behavioural differences observed and recorded.
- Redesigning the Experiment/Run/child-Trace hierarchy or replacing working recording paths without evidence of incompatibility.
- Implementing Fiducia alignment in this phase; its dependency and subsequent work are documented.
- Reopening deferred catalogue, Cognition, Compaction, dynamic extension loading or broad protocol-version negotiation.

## 2. Approved architectural invariants

1. **Execution Profile versus SEP.** An Execution Profile is a user-selectable configuration combining execution mode (AUTO, BUFFERED, STREAMING), Adapter mode (AUTO/CIM or NONE/pass-through), System Prompt, ordered Filters and Tools. The persisted SEP **does not contain or depend upon the originating Execution Profile**; it contains the actual resolved Adapter, Filter, System Prompt and execution configuration required to reproduce the setup.
2. **Original Setup.** Repetere obtains the authoritative recorded SEP from Vestigare and asks Moderari/SIE to reconstruct its resolved execution definition. Neither the originating Execution Profile nor saved prompt/profile definitions are looked up. Deleting or changing them must not prevent reconstruction.
3. **Exactness.** Required executable components, identities, versions and other essential dependencies must be available and compatible. Missing/incompatible requirements produce an explicit, actionable Moderari failure returned to Repetere. **No silent substitution or fallback.**
4. **New Setup.** Repetere behaves as another client: new session via Pontis; provider/model reservation through Praebere; applicable Execution Profiles returned by Moderari via the established client path; profile selection sent to Moderari; first recorded client message triggers SIE SEP materialisation and Vestigare acknowledgement.
5. **Session isolation.** Each replay uses a fresh Pontis-owned session ID, replay-specific immutable SEP, recording and child trace. Original SEP and trace remain immutable. Repetere initiates but does not own authoritative session termination or model activation/unload policy.
6. **SEP authority.** Moderari owns validation/materialisation; Vestigare owns persisted SEP and recording. Moderari's diagnostic export is not a substitute for Vestigare's authoritative persisted SEP.
7. **Evidence separation.** Vestigare records client-boundary conversation and authoritative SEP/trace; Moderari records internal execution/transformation evidence; Repetere owns Experiment/Run semantics, including first divergence and replay outcome.
8. **Replay goal.** Recreate original configuration, not original model behaviour. A different output is valid experimental evidence, not by itself a replay failure.
9. **Execution settings.** Streaming AUTO/BUFFERED/STREAMING and Adapter AUTO/NONE are distinct dimensions; recorded resolved behaviour and exact configuration must be respected. ADAPTERS_DISABLED/pass-through must not imply buffering.
10. **Readiness gate.** No replay First Ask proceeds until session configuration is valid and the SEP is acknowledged by Vestigare according to Moderari's lifecycle.

## 3. Current code baseline and preservation strategy

The supplied Repetere package includes `services/replay_runtime.py`, `infrastructure/trace_control.py`, `infrastructure/nuntius_control.py`, corresponding tests, and existing replay persistence and UI. Moderari includes `sie/session.py`, `sie_integration.py`, `sie_http.py`, `execution_profiles.py` and related tests. Vestigare includes `session_plan.py`, `session_plans.py` and tests. These are inspection targets; **method-level contracts and full test coverage must be verified in Increment 01** rather than assumed from filenames or historical milestone claims.

| Existing area | Default disposition | Verification focus |
|---|---|---|
| Experiment/Run persistence | Retain | Stable IDs, state transitions, failures, correlation |
| Child-trace provenance | Retain | `source_trace_id`, `experiment_id`, `run_id`, session identity |
| Recording loader and message extraction | Retain/adjust | Exact client-boundary requests/responses, message order |
| Replay planner and divergence logic | Retain | First deviation, matched prefix, continuation, forced divergence |
| Replay runtime | Modify at preparation boundary | Replace universal pass-through prerequisite with SEP-aware preparation |
| Pontis/Praebere clients | Adapt | Normal session creation, reservation, activation and termination |
| Moderari control client | Extend | Original Setup reconstruction and New Setup selection |
| Vestigare trace client | Retain/extend minimally | Existing recording operations, SEP linkage, divergence provenance |
| UI/API | Extend | Setup mode, profile selection, validation errors, recovery |
| Fiducia-facing interface | Preserve where possible | Explicit compatibility/versioned changes for later phase |

**Known legacy seam to investigate:** `establish_moderari_pass_through` in Repetere's replay preparation path must not remain the universal prerequisite: replay supports both Adapter-enabled and native pass-through configurations.

## 4. Target workflows

### 4.1 Original Setup

1. Operator selects a source Vestigare trace and requests Original Setup.
2. Repetere loads the original client-boundary conversation and its authoritative persisted SEP; verifies trace/SEP association and integrity information available from Vestigare.
3. Repetere asks Moderari to **preflight validate** exact reconstructability from the resolved SEP. This is a proposed two-stage interface, not yet confirmed in existing code.
4. Moderari validates required Adapter/Filter implementations and exact versions, configuration completeness, protocol/representation constraints, model/provider requirements, execution settings and essential dependencies **without consulting the originating Execution Profile or saved prompt definition**.
5. On failure, Moderari returns structured reasons. Repetere reports the failure and offers **New Setup**; it does not silently alter the configuration.
6. On success, Repetere establishes a new Pontis-owned replay session and arranges the original provider/model reservation through the normal Praebere lifecycle, subject to the reconstruction contract.
7. Moderari creates a **new immutable replay SEP** under the new session ID, with equivalent resolved execution configuration and provenance to the source SEP. Precisely when reconstruction is committed versus First Ask is an Increment 01 contract decision; the preferred design is preflight validation followed by authoritative materialisation at First Ask.
8. Vestigare acknowledges and persists the replay SEP; the replay child trace is associated with Experiment/Run and source trace.
9. Repetere submits the first recorded client message and continues the existing replay/matching/divergence lifecycle.
10. On completion/cancellation/failure, Pontis owns termination; Repetere finalises Run and recording according to existing semantics.

**Critical identity rule:** Do not copy the original session ID into the new authoritative replay session. Preserve source session/SEP identifiers only as provenance. Define a deterministic comparison of *execution-equivalent fields* excluding replay identity and provenance fields.

### 4.2 New Setup

1. Operator selects source trace and New Setup.
2. Repetere creates a fresh Pontis session.
3. Repetere selects provider/model through the established Pontis → Praebere reservation path.
4. Praebere constructs/transmits the EMD to Moderari; Moderari determines applicable **Execution Profiles** and returns the choices through the existing client-facing contract.
5. Repetere displays and selects one applicable Execution Profile; it submits selection to Moderari via the existing control route. No unrequested default selection or silent profile repair.
6. On first replayed client message, SIE resolves execution, materialises a new immutable SEP and obtains Vestigare acknowledgement before execution.
7. Repetere continues the normal replay engine and records independent child trace, Run and provenance. The original SEP remains unchanged.

**Special case:** The absence of semantic Filters must be represented by the applicable profile/configuration (including `None` where supported); it must not bypass the Execution Profile contract or erase independently selected execution/Adapter settings.

### 4.3 Original Setup failure and recovery

- Failure is explicit, diagnostic and attributable to the missing or incompatible dependency.
- No execution or substitution occurs on the failed Original Setup path.
- New Setup remains selectable, using the normal model/profile flow.
- Whether a failed preflight receives a Run record, a preparation-attempt record, or both is **deferred for focused design discussion**. The requirement is to preserve useful failure evidence without inventing a successful replay or unnecessary child trace.
- If a session was already created, authoritative cleanup must occur through Pontis, with idempotent finalisation and no leaked model reservations.

## 5. Associated-service changes and contracts

### 5.1 Moderari / SIE — primary dependency

**Inspect and verify:**

- Persisted SEP schema: resolved Adapter identities/versions/configuration, ordered Filter identities/versions/configuration, literal System Prompt content, execution mode, Adapter mode, model/provider identity, protocol/CIM version and tool requirements.
- Existing reconstruction path, if any; remove any dependence on current Execution Profile registry or saved prompt/profile definitions for Original Setup.
- Exact version resolution, availability and incompatibility checks; explicit missing/unsupported/unknown outcomes.
- Preflight validation semantics versus authoritative materialisation; immutability and concurrency/idempotency.
- Replay SEP provenance and new-session identity handling.
- Vestigare acknowledgement gate and failure propagation.
- ADAPTERS_ENABLED and ADAPTERS_DISABLED, buffered/streaming/AUTO behaviour.

**Expected implementation:** A documented, testable reconstruction/validation contract owned by Moderari. Repetere must not duplicate SIE component resolution or construct private SIE objects.

### 5.2 Vestigare — persistence and recording

**Verify/extend only as needed:**

- Authoritative SEP retrieval by source trace/session after the original session has ended.
- SEP integrity and full resolved configuration persistence.
- Recording of new replay SEP, session, source-trace provenance and child trace.
- Existing Repetere `TraceControlClient` operations: recording start, exchange capture, finalisation, deletion. Confirm whether calls are direct HTTP ingestion/control or other transport; do not replace working endpoints without a contract reason.
- Distinguish actual replay exchanges from Repetere's first-divergence *interpretation*. Determine whether first-divergence metadata remains solely on Run or also needs an optional trace annotation.
- Correct behaviour when recording is disabled for ordinary sessions, noting that SEP acknowledgement remains mandatory.

**No duplicate Moderari internal evidence in Vestigare.**

### 5.3 Pontis — session authority

- New replay session creation and immutable session ID.
- Replay First Ask routing to Moderari, not through legacy Pi inference.
- Cancellation, forced close and session termination; notifications to Moderari, Praebere and relevant client.
- Tool-dispatch continuation through established Rogare/Pi SDK host where applicable.
- Idempotent cleanup on preparation rejection, timeout and partial failure.

### 5.4 Praebere — provider/model authority

- Model reservation bound immutably to replay session; no mid-session model switching.
- EMD handoff and applicable Execution Profile discovery path.
- Lazy activation on First Ask, reference-counted unload on authoritative termination.
- Original Setup must honour recorded model/provider requirements or fail explicitly if they cannot be satisfied; New Setup may choose a different model.

### 5.5 Repetere — principal changes

- Replace universal pass-through preparation with explicit `ORIGINAL_SETUP` / `NEW_SETUP` orchestration.
- Add source SEP retrieval, provenance, preflight request and error presentation.
- Reuse normal session/model/profile selection rather than creating a separate profile-resolution engine.
- Integrate new SEP readiness gate with existing replay runtime.
- Preserve conversation extraction, matched-prefix replay, first-divergence logic, Experiment/Run/child-Trace records and existing trace-control integration.
- Extend UI/API for setup mode, model/profile choice, component-version errors, recovery and status.
- Preserve existing public interfaces used by Fiducia where possible; otherwise provide a documented migration contract.

### 5.6 Rogare and Servire — likely limited changes

- **Rogare:** primarily reference implementation of the working client configuration path; avoid modifying unless a shared contract requires correction.
- **Servire:** only configuration/observability changes proven necessary for replay, without transferring provider/model or session authority.

### 5.7 Fiducia — subsequent phase

Document the stable replay invocation, result, cancellation, failure and provenance contracts for future scheduled replay. Do not implement Fiducia until Repetere's RPR-01–09 gate is met.

## 6. Incremental work packages

### Increment 01 — Source audit, contracts and preparation lifecycle

**Purpose:** Establish exact source-level gaps and approve interfaces before code changes.

**Tasks**

1. Audit Repetere runtime, repositories, API/UI, trace control, session clients, cleanup and Fiducia-facing surface.
2. Audit Moderari SEP schema, materialisation, Execution Profile resolution, component registries and Vestigare acknowledgement.
3. Audit Vestigare SEP persistence/retrieval and replay recording operations.
4. Trace complete existing Rogare/external-client session-selection calls, including Nuntius routing.
5. Specify Original Setup reconstruction preflight and commit/First Ask operations, input/output schemas, errors, idempotency and authority.
6. Specify new replay SEP provenance and execution-equivalence comparison rules.
7. Specify Repetere preparation state machine and session cleanup boundaries.
8. Resolve outstanding failure-evidence placement and preflight/session-creation ordering.
9. Produce a service-by-service change inventory, contract fixtures and test matrix.

**Exit gate:** Approved API/control contracts, ownership/state diagrams, exact file-level change inventory and tests. No unapproved implementation.

### Increment 02 — Original Setup and exact SEP reconstruction

**Tasks**

1. Implement any required Moderari reconstruction/validation API and version-exact resolution.
2. Implement Vestigare SEP retrieval/integrity adjustments only if inspection finds gaps.
3. Implement Repetere source SEP loading and Original Setup preparation.
4. Materialise replay SEP with new session identity and source provenance through Moderari; preserve acknowledgement gate.
5. Exercise success and missing/incompatible component failures; verify deleted/changed saved profile and prompt do not matter.
6. Verify recording and cleanup on failed preflight and materialisation.

**Exit gate:** RPR-03, RPR-07 and RPR-08 proven, plus independent session/SEP identity and immutable original evidence.

### Increment 03 — New Setup through normal client path

**Tasks**

1. Reuse Pontis session creation and Praebere model reservation.
2. Present Moderari's applicable Execution Profiles for the reserved provider/model.
3. Send profile selection through the existing Moderari control contract.
4. On first recorded client message, let SIE materialise the new SEP and obtain Vestigare acknowledgement.
5. Exercise different model, changed System Prompt/Filters/Tools, explicit no-Filter configuration and invalid/stale selections.
6. Confirm no legacy global-model, automatic pass-through or bypass path remains.

**Exit gate:** RPR-05 and RPR-06 proven using the same underlying contracts as Rogare/external clients.

### Increment 04 — Replay engine and child-trace integration

**Tasks**

1. Connect both prepared modes to the existing replay engine without rewriting divergence logic.
2. Preserve source client-boundary request ordering, exact recorded input and tool/result continuation as applicable.
3. Preserve Experiment → Run → child-Trace hierarchy and existing recording lifecycle.
4. Verify first divergence, matched prefix, forced divergence, child trace provenance and session-specific SEP/evidence.
5. Verify completion, cancellation, termination, restart and cleanup paths.
6. Ensure the replay SEP and execution evidence belong to the new session, never the original.

**Exit gate:** RPR-01, RPR-02 and RPR-04 demonstrated, with existing replay regression suite green.

### Increment 05 — Recovery, UI, regression and phase closeout

**Tasks**

1. Expose Original/New Setup choices and validation status in Repetere UI/API.
2. Show actionable component-version errors; offer New Setup after exact reconstruction refusal.
3. Define/persist preparation-failure evidence per the agreed contract.
4. Verify UI state across failure, retry, cancellation, forced-close and refresh.
5. Run full Repetere tests, cross-service contract tests and targeted live replay tests.
6. Update user/operator documentation, requirements register, Engineering Diary and Fiducia handoff specification.
7. Deliver complete synchronised packages for every service changed.

**Exit gate:** RPR-01–09 demonstrated and documented; no outstanding high-severity regressions.

## 7. Proposed preparation state machine

This is a **design proposal**, to be reconciled with existing Repetere state names in Increment 01.

`CREATED → SOURCE_LOADED → VALIDATING → CONFIGURING → WAITING_FIRST_ASK → SEP_READY → REPLAYING → COMPLETED`

Failure/exit branches: `VALIDATION_REJECTED`, `PREPARATION_FAILED`, `EXECUTION_FAILED`, `CANCELLED`, `TERMINATED`.

- `VALIDATION_REJECTED`: explicit Original Setup incompatibility; New Setup offered.
- `PREPARATION_FAILED`: infrastructure/session/recording/selection failure; cleanup as required.
- `WAITING_FIRST_ASK`: configuration selected/validated but no authoritative SEP committed yet.
- `SEP_READY`: Moderari confirms immutable replay SEP and Vestigare acknowledgement.
- Terminal transitions must be idempotent and must not imply a replay occurred if no First Ask executed.

## 8. Contract and provenance data

The following are **conceptual fields**, not asserted existing JSON schemas:

| Record / message | Required concepts |
|---|---|
| Original Setup request | Source trace ID, recorded SEP reference/integrity, requested replay correlation, idempotency key |
| Preflight result | Valid/invalid, exact missing components and versions, compatibility errors, diagnostic code, no substitutions |
| Replay SEP | New session ID, resolved executable configuration, source SEP/trace provenance, materialisation timestamp and integrity |
| New Setup selection | New session ID, reserved provider/model identity, applicable Execution Profile choice, selection acknowledgement |
| Run provenance | Experiment ID, Run ID, child trace ID, source trace ID, original/replay SEP identifiers, setup mode |
| Failure result | Phase, service, structured code, details, cleanup status, recoverability and New Setup eligibility |
| Divergence | First divergent exchange/position, comparison basis, observed outputs, linked Run/child trace |

**Important:** A replay SEP must be execution-equivalent to the recorded SEP for Original Setup, but cannot be byte-identical if it contains the new session ID or replay provenance. The precise canonical comparison rules require approval.

## 9. Verification and acceptance matrix

| ID | Test / acceptance scenario | Expected result |
|---|---|---|
| RPR-01 | Replay from Vestigare client-boundary messages | Inputs preserve source boundary semantics and order |
| RPR-02 | No Filter-transformed model-facing context in source trace | Replay remains possible from recorded client exchanges and SEP |
| RPR-03 | Valid recorded SEP Original Setup | New session with equivalent resolved execution setup |
| RPR-04 | Replay run and child trace | Independent session ID, SEP, evidence, recording and correlation |
| RPR-05 | New Setup with different model/profile | Fresh SEP via normal selection path; source unchanged |
| RPR-06 | Adapter/Filter re-resolution for new model | Moderari resolves only applicable components and configuration |
| RPR-07 | Delete/change saved Execution Profile and prompt definitions | Original Setup succeeds from SEP when executable versions exist |
| RPR-08 | Remove required exact Adapter/Filter version | Explicit refusal, diagnostic returned, no substitution |
| RPR-09 | Original Setup refused | New Setup remains available and works |
| Cross-cutting | ADAPTERS_ENABLED / ADAPTERS_DISABLED | Both supported replay paths execute correctly |
| Cross-cutting | AUTO/BUFFERED/STREAMING and Adapter AUTO/NONE | Independent settings and resolved SEP behaviour preserved |
| Cross-cutting | Vestigare unavailable at SEP acknowledgement | No READY/execution; explicit failure and cleanup |
| Cross-cutting | Duplicate preflight/First Ask/termination | No duplicate SEP, replay, recording or leaked reservation |
| Cross-cutting | Session cancellation / forced close | Pontis authoritative; Repetere and trace state reconcile |
| Cross-cutting | First divergence and forced divergence | Existing semantics and evidence retained |
| Cross-cutting | Tool calls and continuation | Correct host dispatch, correlation and recorded boundary exchanges |
| Cross-cutting | Different model output | Recorded as behavioural divergence, not configuration failure |
| Cross-cutting | Source SEP and original trace immutability | No mutation of original artefacts |

### Test layers

- Unit tests: state machine, canonical SEP comparison, error mapping, replay planning and provenance.
- Contract tests: Repetere ↔ Pontis/Praebere/Moderari/Vestigare; explicit failures and idempotency.
- Integration tests: real SIE materialisation, Vestigare acknowledgement, reservation and child-trace creation.
- Live tests: representative model replay, tools, both Adapter modes and divergence.
- Static gates: `ruff`, `mypy`, `pytest` for each changed service; retain existing regression suites.
- Evidence: capture test IDs, revision, date, expected/actual result, and requirements mapping.

## 10. Risks and controls

| Risk | Control |
|---|---|
| Hidden current-profile lookup during Original Setup | Audit reconstruction code and test deletion of saved definitions |
| Exact component version unavailable | Reject with structured diagnostic; offer New Setup |
| Original/replay session identity leakage | Explicit source vs replay identity fields and tests |
| SEP acknowledged without trace correlation | Verify Vestigare persistence and replay linkage atomically or reconcile failures |
| Model/provider drift or unavailability | Explicit failure for Original Setup; new selection only under New Setup |
| Auto mode resolves differently on later software version | Compare recorded resolved configuration and enforce required fidelity |
| Duplicate First Ask or recording | Idempotency/correlation and replay lifecycle tests |
| Existing divergence logic broken by new preparation | Preserve engine, test matched and divergent paths before/after |
| Cleanup races and leaked reservations | Pontis-owned termination and idempotent compensating cleanup |
| Tools/external side effects during replay | Explicit replay tool policy and test fixtures; do not confuse tool fidelity with model fidelity |
| Over-expanding service changes | Reuse Rogare/client contracts; isolate new API to reconstruction only |

## 11. Open detailed decisions — to settle in Increment 01

These are **not reversals of the agreed architecture**:

1. **Preflight timing:** validate Original Setup before creating a Pontis session where feasible; materialise authoritatively on First Ask. Confirm feasibility and any reservation-dependent checks.
2. **SEP reconstruction transport:** select the exact Moderari HTTP/Nuntius command and payload format after inspecting existing control conventions.
3. **Replay SEP provenance:** define original SEP identifier/hash, source trace and new session correlation fields; decide where stored without modifying original SEP.
4. **Execution-equivalence comparison:** define fields that must match exactly and fields legitimately changed by replay identity/provenance.
5. **Failure evidence:** decide where a failed preflight or SEP materialisation attempt is persisted (Run, preparation attempt, Moderari evidence, or combination), and at what point a Run is allocated.
6. **Divergence metadata:** confirm whether first divergence remains a Repetere Run field only or merits a Vestigare annotation.
7. **Tool availability:** clarify how Original Setup validates required tool execution capabilities and how missing tool hosts are reported.
8. **Execution Profile terminology and API:** align user-facing naming with current Moderari implementations while preserving the agreed meaning.

## 12. Deliverables and package discipline

For every approved implementation increment:

- Complete, synchronised **Repetere code package**, not isolated patches.
- Complete, synchronised package for **each associated service actually changed** (Moderari, Vestigare, Pontis, Praebere, Rogare or Servire).
- Change inventory with affected paths, interface/schema changes and migration notes.
- Updated unit/contract/integration tests and reported `ruff`/`mypy`/`pytest` results; do not claim unrun tests passed.
- Updated architecture/requirements notes, RPR evidence and any remaining known limitations.
- Regression confirmation for Experiment/Run/child-Trace, first divergence, recording, cleanup and existing replay UI.

**Increment 01 deliverable before implementation:** reviewed file-level audit, approved contracts and state machine, final dependency matrix and explicit go/no-go for Increment 02.

## 13. Phase completion and subsequent sequence

**Repetere phase exit:** RPR-01–09 have dated evidence; both Adapter-enabled and pass-through replay tested; Original Setup exactness and rejection demonstrated; New Setup through normal client path demonstrated; original artefacts immutable; new session/SEP/child trace independent; existing replay functionality retained.

After this phase, proceed in the agreed order:

1. Fiducia alignment with the new Repetere invocation and failure contract.
2. Cross-service architecture stabilisation.
3. Outstanding and deferred development decisions and implementation.
4. Formal verification of all in-scope requirements.
5. `NEXT` release build.

## 14. Source basis and evidence limitations

- `LUMEN-Requirements-Status-and-Development-Roadmap-2026-10-31(1).md`: governing sequence, RPR-01–09, ownership and phase gates.
- `ENGINEERING_DIARY(20261031-183947).md`: historical design rationale, October SIE/SEP integration and replay evolution; historical statements may be superseded by later decisions.
- `Lumen-Repetere-0.20.20-Failed-Preparation-Lifecycle-Fix-QA(1).zip`: implementation baseline.
- `Lumen-Moderari-Full-2026-10-30-Session-Evidence-Export-Mypy-Fix(1).zip`: SIE/SEP and profile implementation baseline.
- `Lumen-Vestigare-Session-Cleanup-20261018(3).zip`: SEP persistence and trace integration baseline.
- Explicit architectural decisions made in the 31 October 2026 discussion supersede conflicting legacy terminology or proposals.

This is a **planning document**, not a completed line-by-line code audit or verified implementation report. The code-level interface audit, including whether Moderari already has a reusable reconstruction facility, is the first deliverable of Increment 01.
