# Lumen Current Status

**Document status:** Current snapshot  
**Snapshot date:** 2026-11-04  
**Evidence baseline:** Lumen-Community repository snapshot of 2026-11-03; Engineering Diary through 2026-10-31  
**Purpose:** Concise, maintained account of current engineering state; not a release certification

## What Lumen Is Today

Lumen is an independently developed, model- and client-independent **Reasoning Assurance Service**. Its focus is preserving execution evidence, reproducing model interactions under controlled conditions and providing a foundation for behavioural assessment. Its operating principle is **Observe → Reproduce → Repeat**.

The **M0.1 external research distribution is the completed baseline**. Development of the next research build (provisionally **NEXT**, on `research-foundation-next`) is ongoing. The current priority is alignment of replay and scheduled replay with the revised session-owned, immutable Session Execution Plan (SEP) architecture—not a claim that behavioural assessment is already delivered.

For authoritative architecture, see [Canonical Architecture](architecture/core/ARCHITECTURE.md), [Ecosystem Architecture](architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md), and [Service Matrix](architecture/ecosystem/LUMEN_SERVICE_MATRIX.md).

## Service Status and Responsibilities

| Service | Role | Status / qualification |
| --- | --- | --- |
| **Servire** | Operational control plane and service-group lifecycle | Established M0.1 foundation; NEXT alignment and verification remain |
| **Nuntius** | Shared command, status and diagnostic path | Established foundation; cross-service integration remains subject to verification |
| **Rogare** | Research/session interface and current Pi SDK tool host | Active integration and tool-dispatch testing |
| **Pontis** | Authoritative client session lifecycle, cancellation and tool routing | Revised session/tool routing implemented or under integration verification |
| **Praebere** | Provider discovery, model reservation, readiness and Lumen-initiated residency | Session-owned reservation and lazy activation developed; broader conformance remains |
| **Moderari** | Orchestration, Session Interaction Engine, adapter/filter resolution and immutable SEP | Substantial implementation and integration evidence; full protocol/streaming conformance not certified |
| **Vestigare** | Correlated trace evidence and persisted execution-plan acknowledgement | Active evidence path; replay integration and failure-path verification remain |
| **Repetere** | Experiments, runs and fresh-session replay | **Current NEXT alignment priority**: Original Setup from recorded SEP and New Setup from newly resolved execution configuration |
| **Fiducia** | Scheduled and repeated replay workflows | M0.1 foundation exists; NEXT contract alignment follows Repetere |
| **Aestimare** | Behavioural assessment and evidence synthesis | **Research/design direction; not included in M0.1**; not represented as a completed assessment engine |
| **Audire** | Proposed multi-host operational transport | Proposed; not part of the implemented critical path |

Status labels reflect documented development and integration evidence, **not a fresh source-code audit or formal release acceptance**. Consult the canonical service matrix for current ownership and dependencies.

## Execution Architecture Under Development

The NEXT architecture separates three concerns:

1. **Reservation:** Pontis owns the session; Praebere reserves the selected provider/model for that session. Selection is not model loading.
2. **Planning:** Moderari's Session Interaction Engine resolves execution configuration into an immutable **Session Execution Plan (SEP)** on First Ask. Vestigare acknowledgement is required before READY, independently of whether conversation recording is enabled.
3. **Activation and execution:** Praebere manages model readiness and residency; Moderari sends inference requests to the selected provider. Tool execution is routed through Pontis to the current Rogare Pi SDK host.

For **Original Setup** replay, Repetere reuses the recorded SEP, rebound to a fresh session, with Moderari responsible for validation. For **New Setup**, a fresh configuration is resolved and materialised. Repetere does not reinterpret recorded filters, adapters or system prompts. These are current design and integration targets; their full end-to-end verification is not asserted here.

## Current Development Sequence

The Engineering Diary's 31 October roadmap establishes this order:

1. Repetere alignment with immutable SEP and trace provenance.
2. Fiducia alignment with the revised replay contract.
3. Cross-service architecture stabilisation.
4. Outstanding and deferred requirements.
5. Formal cross-service verification and acceptance evidence.
6. NEXT release build and packaging.

Implementation-level tests occur throughout; **formal verification and release acceptance are future gates**. The `main` branch retains the M0.1 baseline until the revised architecture is ready.

## Research and Evidence

The [Selected Experimental Investigations](research/SELECTED_EXPERIMENTAL_INVESTIGATIONS.md) index provides a short route into the [Engineering Diary](engineering/ENGINEERING_DIARY.md). Documented investigations include long-context continuity, checkpoint prompt changes with a negative result, and tool-execution evidence showing why a correct output is not the same as complete procedural compliance.

These are engineering experiments and observations, not independently validated claims about internal model cognition. In particular, text checkpoints are **observable artefacts**; they do not provide direct access to a model's hidden internal state.

## Deferred or Unverified

- Aestimare behavioural assessment and specialist assessors remain future research/development work.
- Full protocol and streaming conformance, terminal-outcome taxonomy and edge-case tool failures require additional verification or implementation.
- Persistent model catalogue/user overrides, advanced filters and broader extension support remain in the tracked deferred scope.
- Audire multi-host transport and an external contributor setup process are not current critical-path capabilities.

Refer to the current requirement status register and canonical service documents for detailed scope; this summary is not an exhaustive requirement inventory.

## Documentation and Historical Status

The August repository reorganisation and documentation-overlap review are preserved as historical records. Current architectural authority belongs to canonical documents; the [Engineering Diary](engineering/ENGINEERING_DIARY.md) preserves dated observations, including decisions subsequently revised. The [Documentation Index](README.md) provides navigation.

## Maintaining This Page

Update the snapshot date when the actual state is reviewed. Record major changes here, update the canonical service matrix and service documentation as necessary, and retain historical evidence in the Engineering Diary. **Do not silently promote planned work to implemented status.**
