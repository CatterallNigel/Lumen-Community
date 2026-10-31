# Lumen — Consolidated Requirement Status Register and Development Roadmap

**Baseline:** 31 October 2026 · branch `research-foundation-next` · next release placeholder `NEXT`
**Inputs:** All 14 documents in `LUMEN-DOCUMENTED_REQ_CHANGES.zip`, plus the established October 2026 integration-test outcomes.
**Nature:** Planning and traceability register; **not** a fresh code audit of every repository.

## 1. Governing development sequence

1. **Repetere** — bring replay to the immutable SEP / SIE architecture; Original Setup, New Setup, failure paths and replay provenance.
2. **Fiducia** — align scheduled replay with the new Repetere contract and prove orchestration.
3. **Cross-service architecture stabilisation** — bring Moderari, Praebere, Pontis, Rogare, Vestigare and Servire into consistent new-architecture operation; address regressions and interface gaps.
4. **Return to deferred and incomplete development** — catalogue and user EMD overrides; Cognition, Compaction; dynamic extension loading; multi-version negotiation; full Adapter/CIM semantic and streaming coverage; any other deferred work in the register. Resolve priorities and dependencies within this phase.
5. **Formal verification phase** — systematically verify every requirement and acceptance criterion, including previously implemented and partially implemented items; fix failures and close evidence gaps.
6. **Release build** — version freeze, Linux/Docker packaging, security and integration gates, release candidate and release.

**Release sequencing decision:** verification is the final formal gate, but unit/integration tests are required continuously during development. A feature cannot be called Implemented solely because a document planned it.

## 2. Status vocabulary and confidence

| Status | Interpretation |
|---|---|
| Implemented | Existing implementation and/or credible recent integration evidence; still subject to final verification gate. |
| Partially Implemented | Some functions exist, but declared scope or acceptance remains incomplete. |
| Verification Required | Design/implementation is plausible or reported, but current acceptance evidence is insufficient. |
| Not Started | No verified implementation of the **new-architecture requirement**; legacy code may exist. |
| Deferred | Explicitly postponed, retained in the roadmap rather than dropped. |

**Evidence qualifier:** source documents often describe historical milestones; October 2026 live tests support Qwen tool cycles, session flow and evidence export. Where the code of another service was not inspected, status is conservative and does not imply that no code exists.

## 3. Consolidated requirement register

The **Source** column uses the document key in §6. Acceptance criteria are testable targets, not claims that tests already passed.

| ID | Area | Status | Source | Acceptance criterion / outstanding detail |
|---|---|---|---|---|
| `MOD-01` | Moderari | Implemented | IP,SEP | Fixtures and schema/mapping checks execute deterministically across supported cases. — IP-01.1 conformance harness |
| `MOD-02` | Moderari | Implemented | IP,SEP | IN/OUT enforce declared schema, CIM and semantic-loss contracts. — 5A BaseAdapter |
| `MOD-03` | Moderari | Implemented | IP,SEP | Both protocol adapters pass baseline production conformance tests. — 5B OpenAI and Anthropic adapters |
| `MOD-04` | Moderari | Implemented | IP,SEP | Exact, family, base fallback and ambiguity cases resolve deterministically. — 5C/5C.1 adapter resolution |
| `MOD-05` | Moderari | Implemented | IP,SEP | Declared textual tool calls become structured tool calls; mixed prose retained; invalid batches rejected atomically. — 5D/5D.1 Qwen tool SpAd |
| `MOD-06` | Moderari | Implemented | IP,SEP | System instructions are inserted/replaced according to Filter configuration and order. — 5E System Prompt Filter |
| `MOD-07` | Moderari | Verification Required | IP,SEP | Registry discovery identifies and rejects conflicting/invalid components. — SIE component discovery |
| `MOD-08` | Moderari | Implemented | IP,SEP | First Ask resolves and freezes SEP; execution follows enabled/disabled mode. — SIE session integration |
| `MOD-09` | Moderari | Implemented | IP,SEP | SEP captures resolved component versions/configuration and cannot change after finalisation. — SEP immutable snapshot |
| `MOD-10` | Moderari | Implemented | IP,SEP | Moderari READY occurs only after Vestigare SEP acknowledgement, regardless of recording switch. — Vestigare SEP acknowledgement |
| `MOD-11` | Moderari | Verification Required | IP,SEP | Same-protocol requests retain native behaviour, including streaming where supported. — ADAPTERS_DISABLED native pass-through |
| `MOD-12` | Moderari | Implemented | IP,SEP | Profiles are selected before First Ask and None is a valid profile. — Filter applicability and explicit None |
| `MOD-13` | Moderari | Verification Required | IP,SEP | Filter execution resumes by filter_execution_id without duplication or cross-session confusion. — Private Filter Ask correlation |
| `MOD-14` | Moderari | Verification Required | IP,SEP | Ordered to_model filters receive and return declared-protocol Context in correct sequence. — Filter order and Context |
| `MOD-15` | Moderari | Deferred | SIE,FIL,OUT,5S | Separate deadline, cancellation, provider and tool failures in evidence and client outcomes. — Execution outcome taxonomy |
| `MOD-16` | Moderari | Implemented | SIE,FIL,OUT,5S | Newest captured evidence appears first, with millisecond timestamps and unchanged sequence numbers. — Evidence UI descending and ms |
| `MOD-17` | Moderari | Implemented | SIE,FIL,OUT,5S | One export includes session SEP and complete raw Extended JSON Mongo evidence, newest-first. — Session SEP + Mongo export |
| `MOD-18` | Moderari | Verification Required | IP,SEP | Unavailability is explicit; authoritative persisted SEP remains retrievable for replay via Vestigare. — SEP export after session release |
| `MOD-19` | Moderari | Deferred | SIE,FIL,OUT,5S | Evidence distinguishes missing JSON paths from literal null values. — Null versus absent evidence values |
| `MOD-20` | Moderari | Verification Required | CIM,CONF,COV,5PF,5P1,5P2 | Full declared semantic and round-trip conformance passes before freezing CIM. — CIM 1.0 freeze gate |
| `MOD-21` | Moderari | Partially Implemented | CIM,CONF,COV,5PF,5P1,5P2 | Every field in declared OpenAI/Anthropic profiles has IN and same-protocol OUT fixtures; gaps explicitly enumerated. — 5P non-streaming semantic coverage |
| `MOD-22` | Moderari | Partially Implemented | CIM,CONF,COV,5PF,5P1,5P2 | Message, image, audio/video/document, Anthropic blocks and usage fields have explicit mappings or declared unsupported status. — 5P nested/multimodal variants |
| `MOD-23` | Moderari | Partially Implemented | CIM,CONF,COV,5PF,5P1,5P2 | Streaming deltas/events, tool argument assembly, usage and stop semantics pass same/cross protocol conformance. — 5Q streaming semantics |
| `MOD-24` | Moderari | Verification Required | CIM,CONF,COV,5PF,5P1,5P2 | Unrepresentable destination semantics yield PASS_WITH_SEMANTIC_LOSS and persisted final evidence. — Cross-protocol OUT semantic loss |
| `MOD-25` | Moderari | Verification Required | CIM,CONF,COV,5PF,5P1,5P2 | Preserved source-specific fields cannot be rebound to wrong message/array element. — Opaque extension rebinding |
| `MOD-26` | Moderari | Verification Required | CIM,CONF,COV,5PF,5P1,5P2 | Moderari Mongo evidence captures only final OUT losses, following corrected 5S.1 ownership. — Final OUT semantic loss persistence |
| `MOD-27` | Moderari | Verification Required | SIE,FIL,OUT,5S | Unsupported and unknown capabilities are distinct; required constraints reject deterministically. — Provider/model compatibility constraints |
| `MOD-28` | Moderari | Deferred | CIM,CONF,COV,5PF,5P1,5P2 | Multiple CIM and protocol versions can be selected and validated with explicit compatibility rules. — Protocol/CIM multi-version negotiation |
| `MOD-29` | Moderari | Deferred | SIE,FIL,OUT,5S | Security, provenance, manifest validation and isolation model defined before dynamic loading. — Arbitrary third-party extension loading |
| `MOD-30` | Moderari | Deferred | SIE,FIL,OUT,5S | Separate observational/assessment Filter specification and tests approved. — Cognition Filter |
| `MOD-31` | Moderari | Deferred | SIE,FIL,OUT,5S | Compaction preserves declared Context semantics with auditable transformations. — Compaction Filter |
| `MOD-32` | Moderari | Verification Required | SIE,FIL,OUT,5S | Regression documents deterministic structural conversion may execute valid illustrative blocks; accepted policy is explicit. — Qwen textual example risk |
| `PBR-01` | Praebere | Implemented | COMP §19 | Model/provider authority belongs to Praebere. |
| `PBR-02` | Praebere | Implemented | COMP §19 | Minimal provider/model identity and attributes available. |
| `PBR-03` | Praebere | Implemented | COMP §19 | Unknown characteristics represented explicitly, not as unsupported. |
| `PBR-04` | Praebere | Deferred | COMP §19 | Persistent catalogue supports curated definitions. |
| `PBR-05` | Praebere | Deferred | COMP §19 | User definitions/overrides supported without editing code. |
| `PBR-06` | Praebere | Deferred | COMP §19 | Built-in updates do not overwrite user-owned definitions. |
| `PBR-07` | Praebere | Partially Implemented | COMP §19 | Effective characteristic provenance visible for every value. |
| `PBR-08` | Praebere | Partially Implemented | COMP §19 | One effective EMD assembled with agreed precedence; discovery-only phase explicit. |
| `PBR-09` | Praebere | Verification Required | COMP §19 | Incomplete models remain selectable where requirements permit. |
| `PBR-10` | Praebere | Verification Required | COMP §19 | Praebere does not own Moderari Filter/Adapter semantics. |
| `PBR-12` | Praebere | Deferred | COMP §19 | Exactly one user profile per model is enforced. |
| `PBR-13` | Praebere | Partially Implemented | COMP §19 | Discovery used first, then user knowledge when implemented. |
| `PBR-14` | Praebere | Verification Required | COMP §19 | Availability and metadata completeness are independent. |
| `PBR-15` | Praebere | Partially Implemented | COMP §19 | Missing model characteristics are visible and editable in intended phase. |
| `PBR-16` | Praebere | Implemented | COMP §19 | Reservation binds provider/model immutably per session. |
| `PBR-17` | Praebere | Verification Required | COMP §19 | Concurrent reservations do not corrupt resident/loading/unloading state. |
| `PBR-18` | Praebere | Implemented | COMP §19 | Reserved model yields EMD and execution definition handoff. |
| `PBR-19` | Praebere | Implemented | COMP §19 | Model activation deferred until First Ask and readiness gating respected. |
| `PBR-20` | Praebere | Verification Required | COMP §19 | Session end unloads model only when no other session needs it. |
| `PBR-21` | Praebere | Implemented | COMP §19 | Applicable Filter Profiles reach client via Praebere. |
| `RGR-01` | Rogare | Implemented | COMP §20 | Model selected before Filter Profile. |
| `RGR-02` | Rogare | Implemented | COMP §20 | Applicable profiles recomputed after reservation. |
| `RGR-03` | Rogare | Implemented | COMP §20 | Only applicable choices presented, including None. |
| `RGR-04` | Rogare | Verification Required | COMP §20 | Unavailable profiles cannot be silently selected or repaired. |
| `RGR-05` | Rogare | Implemented | COMP §20 | Client session exists before model selection. |
| `RGR-06` | Rogare | Implemented | COMP §20 | No First Ask before complete session configuration. |
| `RGR-07` | Rogare | Implemented | COMP §20 | Profile selection delivered directly to Moderari via control channel. |
| `RGR-08` | Rogare | Implemented | COMP §20 | Input readiness reflects reservation and profile selection. |
| `RGR-09` | Rogare | Verification Required | COMP §20 | Filter administration remains Moderari-owned. |
| `RGR-10` | Rogare | Verification Required | COMP §20 | Authoritative server revalidation rejects stale profile selection. |
| `RGR-11` | Rogare | Verification Required | COMP §20 | No silent fallback if profile or tools unavailable. |
| `PTR-01` | Pontis | Implemented | COMP §21 | Pontis owns session identity and lifecycle. |
| `PTR-01A` | Pontis | Implemented | COMP §21 | Session exists before model selection. |
| `PTR-02` | Pontis | Implemented | COMP §21 | Reservation mediated through Praebere. |
| `PTR-03` | Pontis | Implemented | COMP §21 | First Ask triggers lazy model activation. |
| `PTR-04` | Pontis | Verification Required | COMP §21 | No repeated model activation during active session. |
| `PTR-05` | Pontis | Implemented | COMP §21 | No global model authority in Pontis. |
| `PTR-06` | Pontis | Verification Required | COMP §21 | Termination notifications reach Moderari, Praebere and Rogare. |
| `PTR-07` | Pontis | Implemented | COMP §21 | Pontis does not choose model load/unload policy. |
| `PTR-08` | Pontis | Implemented | COMP §21 | First Ask/model execution goes to Moderari, tool dispatch to Rogare. |
| `PTR-09` | Pontis | Verification Required | COMP §21 | Pontis does not infer client protocol/representation. |
| `VTR-01` | Vestigare | Verification Required | COMP §22 | Verify record exact client-boundary request. |
| `VTR-02` | Vestigare | Verification Required | COMP §22 | Verify record exact client-visible response. |
| `VTR-03` | Vestigare | Verification Required | COMP §22 | Verify trace independent of Filter-transformed context. |
| `VTR-04` | Vestigare | Verification Required | COMP §22 | Verify trace independent of Adapter internal representations. |
| `VTR-05` | Vestigare | Verification Required | COMP §22 | Verify correlate sessions, interactions and replay child traces. |
| `VTR-06` | Vestigare | Verification Required | COMP §22 | Verify record generic execution metadata without Moderari transformation evidence duplication. |
| `RPR-01` | Repetere | Verification Required | COMP §24; SEP §18, §20.4 | Use Vestigare client-boundary conversation as replay input. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-02` | Repetere | Verification Required | COMP §24; SEP §18, §20.4 | Never require model-facing Filter-transformed context in trace. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-03` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | Load recorded self-contained SEP; ask Moderari to establish exact Original Setup before replay First Ask. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-04` | Repetere | Verification Required | COMP §24; SEP §18, §20.4 | Create new replay session, SEP, evidence and child trace. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-05` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | New Setup uses normal model + Filter Profile/None selection, never edits original SEP. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-06` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | Re-resolve Adapter, Filter applicability and effective configuration for new model. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-07` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | Original Setup works after saved prompt/profile changes or deletion. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-08` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | Missing exact Filter/Adapter version rejects Original Setup without substitution. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `RPR-09` | Repetere | Not Started | COMP §24; SEP §18, §20.4 | After Original Setup rejection, New Setup remains available. — Existing Repetere code not audited; Not Started denotes no verified new-architecture implementation, not proof of no code. |
| `SVR-01` | Servire | Verification Required | COMP §24A | Moderari duplicate-registry startup failure triggers full stack rollback. |
| `SVR-02` | Servire | Verification Required | COMP §24A | Servire never auto-selects or disables conflicting extensions. |
| `RTH-01.0` | Rogare/Pontis | Verification Required | RTH §5, §7 | Pi SDK version, licensing and tool-only contract documented and checked on Windows/Linux. |
| `RTH-01.1` | Rogare/Pontis | Implemented | RTH §5, §7 | Rogare hosts real SDK read/write/edit/bash tool execution. |
| `RTH-01.2` | Rogare/Pontis | Implemented | RTH §5, §7 | Pontis relays tool requests and results without performing tools. |
| `RTH-01.3` | Rogare/Pontis | Implemented | RTH §5, §7 | Moderari SEP/tool continuation runs multiple actual cycles. |
| `RTH-01.4` | Rogare/Pontis | Verification Required | RTH §5, §7 | Forced-close notification and in-flight tool cancellation tested. |
| `RTH-01.5` | Rogare/Pontis | Partially Implemented | RTH §5, §7 | Windows happy path proven; errors, duplicate results and cancellations covered. |
| `RTH-01.6` | Rogare/Pontis | Deferred | RTH §5, §7 | Docker/Linux packaging and security controls verified. |
| `FID-01` | Fiducia | Not Started | COMP §25 | Scheduled replay invokes Repetere using its new Original/New Setup contract and correlates resulting evidence. — Explicitly after Repetere. |
| `FID-02` | Fiducia | Verification Required | COMP §25 | Define scheduled replay variation of model, Filter and prompt only after replay semantics are stable. — May be deferred to advanced replay phase if not release-critical. |
| `REL-01` | Release | Not Started | RTH §5.6; COMP §5 | Linux Docker runtime, packaging and startup/shutdown operate as a reproducible deployment. |
| `REL-02` | Release | Not Started | CONF §13; RTH §7 | Run integrated acceptance suite for session lifecycle, tools, replay, scheduling and evidence. |
| `REL-03` | Release | Not Started | IP §9; 5P2 | Document supported protocol coverage and explicit limitations; freeze release baseline. |
| `REL-04` | Release | Verification Required | COMP §24A; RTH §4.3 | Test rollback, tool permissions, failure isolation, cancellation and restart recovery. |

## 4. Phase exit gates

### Phase 1 — Repetere
- RPR-01–09 demonstrated against recorded traces and SEP, including exact-version refusal, deleted/changed saved configuration, explicit None and recovery through New Setup.
- New replay has independent session ID, trace, SEP and evidence; original artefacts remain immutable.
- Both ADAPTERS_DISABLED and ADAPTERS_ENABLED replay cases exercised.

### Phase 2 — Fiducia
- Scheduler invokes the new replay path, records outcomes and correlations, and handles unavailable recorded components without silent substitution.
- Scheduling, cancellation, failure and retry/idempotency behaviours tested.

### Phase 3 — Architecture stabilisation
- All services agree on session lifecycle, tool continuation, ownership and error contracts.
- Any incompatible legacy paths removed or explicitly isolated; baseline integration tests green.

### Phase 4 — Deferred backlog implementation
- Every Deferred/Partially Implemented row has an explicit decision: implement now, retain as documented future release, or close with approved scope change.
- The currently deferred items are **not lost**; catalogue, Cognition, Compaction, extension loading and multi-version negotiation stay visible.
- Full protocol coverage is staged by declared support profile rather than falsely claiming exhaustive support.

### Phase 5 — Formal verification
- All in-scope rows have dated evidence: test ID, run result, source revision and responsible service.
- Ruff, mypy, pytest, contract/conformance tests, live tool and replay tests, startup failure and persistence tests pass.
- Remaining exclusions are explicitly approved and recorded as release limitations.

### Phase 6 — Release build
- Reproducible deployment, startup rollback, recovery, package manifests, supported-platform documentation and release acceptance signed off.
- Freeze release code only after the formal verification gate.

## 5. Open design and documentation reconciliations

1. **5S.1 evidence ownership:** newer `APPLY-5S.1.md` overrides older wording that would persist final Moderari transformation evidence in Vestigare. Moderari stores execution evidence; Vestigare stores authoritative SEP/trace.
2. **5P coverage:** a passing fixture suite does not certify complete OpenAI/Anthropic field coverage; `5P2` explicitly says IN PROGRESS / NOT CERTIFIED.
3. **RTH workspace guidance:** the RTH plan suggests path containment; subsequent architectural decision permits arbitrary absolute paths subject to OS/container permissions. Reconcile policy explicitly, especially for `bash`, before release.
4. **Tool execution and model fidelity:** a successful tool response is not proof of correct task content. Keep artifact assertions separate from tool-dispatch assertions.
5. **SEP retrieval:** Moderari diagnostic export may report SEP unavailable after process/session release; Repetere must use Vestigare’s persisted authoritative SEP.
6. **Repetere status:** the requirements are precise, but no Repetere source was included in the 14-document archive. Verify any legacy or partial implementations when phase 1 starts.
7. **Deferred release scope:** the requested sequence returns to deferred development before formal verification/release. For large areas such as exhaustive protocol coverage, explicitly decide the supported release profile and which enhancements remain future milestones.

## 6. Source document index

- **SIE** — `01_MODERARI_SIE_ARCHITECTURE.md`
- **CIM** — `02_MODERARI_CIM_ADAPTER_ARCHITECTURE.md`
- **FIL** — `03_MODERARI_FILTER_ARCHITECTURE.md`
- **IP** — `04_MODERARI_IP-01_IMPLEMENTATION_AND_STATUS.md`
- **CONF** — `05_MODERARI_CONFORMANCE_SPECIFICATION.md`
- **COMP** — `06_LUMEN_COMPANION_SERVICE_CHANGES.md`
- **5S** — `APPLY-5S.1.md`
- **COV** — `LUMEN_MODERARI_ADAPTER_RESPONSE_PROTOCOL_COVERAGE_BACKLOG.md`
- **OUT** — `Lumen_Moderari_OUT_Materialisation_Semantic_Loss_and_Specialised_Adapter_Architecture_02026-10-10.md`
- **SEP** — `Lumen_Moderari_SIE_Contract_and_Session_Execution_Plan.md`
- **RTH** — `Lumen_RTH-01_Rogare_Tool_Hosting_Implementation_Plan.md`
- **5PF** — `MILESTONE-5P-PROTOCOL-COMPATIBILITY-FOUNDATION.md`
- **5P1** — `MILESTONE-5P-SEMANTIC-COVERAGE-IMPLEMENTATION.md`
- **5P2** — `MILESTONE-5P-SEMANTIC-COVERAGE-IMPLEMENTATION-PART-2.md`

## 7. Register maintenance protocol

- On every merged work package, update status, evidence link/test, service revision, owner and date; do not infer completion from a design document.
- Mark **Implemented** only when code exists and targeted tests pass; mark **Verification Required** when integration or conformance is still unproven.
- Preserve requirement IDs from the companion specification; `MOD-*`, `FID-*`, and `REL-*` are tracking IDs introduced by this register for otherwise unnumbered requirements.
- Keep deferred items in this same register and the Phase 4 roadmap; do not remove them merely because they are outside the immediate Repetere/Fiducia work.
