# Lumen Project Status and Roadmap
**Date:** 2026-08-10  
**Status:** Active development

## 1. Current Direction

Lumen is now defined as the name of the complete reasoning-assurance ecosystem rather than the name of a single orchestrator.

The existing orchestration component currently called **Lumen** will be renamed **Lumen Moderari**. Each major component within the ecosystem has a distinct responsibility and identity while remaining part of the Lumen family.

This reinforces a central architectural principle:

> **Each Lumen component should have one clearly bounded responsibility.**

The immediate development programme is therefore not simply feature expansion. It is also a deliberate separation and clarification of responsibilities between the components.

---

## 2. Lumen Component Family

| Component | Role |
|---|---|
| **Lumen Moderari** | Core model orchestration, continuity and reasoning-flow management |
| **Lumen Servire** | Operational control plane and unified component workspace |
| **Lumen Pontis** | Client/session and protocol bridge |
| **Lumen Vestigare (Trace)** | Behaviour and execution recording |
| **Lumen Repetere (Replay)** | Deterministic execution of a single replay experiment |
| **Lumen Rogare** | User-facing conversational/client interface into the Lumen stack |
| **Lumen Fiducia** | Evidence-driven replay orchestration, scheduling and calibrated trust |
| **Lumen Aestimare (Assess)** | Behavioural assessment and sufficiency evaluation |

**Lumen** itself is the family and complete stack.

---

## 3. Immediate Priority — Architecture and Naming Standardisation

Before development of Rogare begins, the existing codebase should be standardised around the revised component model.

### 3.1 Rename the Existing Orchestrator

The component presently called simply `Lumen` becomes:

**Lumen Moderari**

The change should be reflected consistently across:

- application/service naming;
- configuration;
- Servire;
- documentation;
- operational labels;
- UI navigation;
- component metadata;
- logs and status reporting;
- deployment/service definitions where applicable;
- branding and logo treatment.

The intention is not to change Moderari's underlying responsibility during this work. It is an architectural and naming clarification that makes room for Lumen to represent the complete platform.

### 3.2 Servire Component Navigation Standardisation

This work should follow the **Lumen → Lumen Moderari** rename. The Moderari rename establishes the component identity first; Servire should then be standardised around that completed naming model.

Servire remains the operational control plane. It should expose the individual Lumen components rather than treating the orchestrator as synonymous with the complete platform.

Top-level component navigation should use one consistent convention:

> **Latin component name — English responsibility**

The intended presentation is:

| Component | English responsibility | Servire presentation |
|---|---|---|
| **Servire** | Service | **Servire — Service** |
| **Moderari** | Orchestrator | **Moderari — Orchestrator** |
| **Repetere** | Replay | **Repetere — Replay** |
| **Vestigare** | Trace | **Vestigare — Trace** |
| **Pontis** | Bridge | **Pontis — Bridge** |
| **Rogare** | Console | **Rogare — Console** |
| **Aestimare** | Assess | **Aestimare — Assess** |
| **Fiducia** | Trust | **Fiducia — Trust** |

This convention should be applied consistently to navigation, page headings, component status, lifecycle controls, configuration, dependency displays and operational logging where component identity is shown.

Existing top-level entries such as **Lumen Operations** and **Lumen Checkpoints** should not remain separate component-level navigation items. They are capabilities of Moderari and should sit within the **Moderari — Orchestrator** workspace, for example as Operations and Checkpoints views.

This establishes a Servire UI invariant:

> **Top-level Servire navigation represents Lumen components. Component-specific capabilities belong inside that component's workspace.**

### 3.3 Port Alignment

The current planned service ports are:

| Component | Port |
|---|---:|
| Pontis | `11435` |
| Moderari | `11436` |
| Repetere | `11437` |
| Vestigare | `11438` |

Vestigare/Trace is being moved from `11435` to `11438`, freeing `11435` for Pontis.

---

## 4. Pontis — Current Integration Priority

Pontis provides the bridge between external clients/tool providers and the Lumen stack.

Its architectural invariant remains:

> **Pontis may terminate requests in standalone development and testing, but in normal Lumen operation it does not generate conversational responses. OpenAI-compatible client traffic is transparently forwarded into the downstream Lumen stack.**

Pontis supports two distinct communication modes:

### HTTP Proxy Mode

For client/Pi-initiated traffic:

`Client → Pi → Pontis → Lumen stack`

Pontis is a transparent intermediary and does not create conversational responses.

### ACP Mode

For Lumen-originated conversations requiring Pi's tools:

`Rogare / Lumen → Pontis ↔ Pi`

Pontis establishes and maintains the ACP relationship with Pi while presenting the appropriate OpenAI-compatible interface toward the Lumen stack.

ACP and HTTP modes must remain session-distinct and must not be mixed within a single session.

### Near-Term Pontis Work

- complete Servire integration;
- complete Trace port migration;
- verify end-to-end stack operation;
- preserve configurable downstream routing;
- retain strict separation between HTTP and ACP session modes;
- continue treating Pontis as protocol/session infrastructure rather than an intelligence or orchestration layer.

---

## 5. Rogare — Next New Component

After the current naming/Servire/Pontis standardisation, development moves to **Lumen Rogare**.

Rogare becomes the conversational client of the Lumen ecosystem.

It should behave like an external client rather than being embedded directly into the orchestration layer.

Conceptually:

`Rogare → Pontis → Vestigare → Repetere/Moderari → Model`

where the exact active path depends on the operation being performed.

### Rogare Principles

- Rogare does not own tools.
- Rogare does not execute tools.
- Rogare should not require knowledge of the concrete tool provider.
- Rogare submits conversational requests into Pontis as a client.
- When tools are required, Pontis manages the appropriate ACP relationship with Pi or another future tool provider.
- Rogare should remain replaceable by another client.

### Tool Discovery

On the first Rogare message requiring a tool-capable session, Pontis should establish an ACP session with Pi and obtain the available tool definitions.

Those tools may then be declared to the model even though Rogare itself owns none of them.

When the model requests a tool, Pontis/session infrastructure must ensure that the request is routed to the actual tool provider rather than Rogare.

### Future Investigation

Cross-vendor tool equivalence remains an open architectural subject.

Different providers may expose semantically equivalent tools under different:

- names;
- schemas;
- parameter structures;
- capabilities.

Lumen should not assume universal tool naming. A future normalisation/capability abstraction may be required.

---

## 6. Repetere — Responsibility Clarification

Repetere should remain deliberately small in responsibility.

Its job is:

> **Execute one replay experiment per invocation and observe whether behaviour follows the recorded path.**

Repetere should **not** own scheduling or repeated execution.

The previous requirement that Replay should support concepts such as:

- run this replay X times;
- periodically repeat this replay;
- schedule replay execution;

should be removed from the Repetere specification.

Those concerns belong to Fiducia.

This keeps Repetere deterministic and focused:

`Replay definition → one execution → replay result`

---

## 7. Fiducia — Move Before Assess

The development sequence is now explicitly:

**Rogare → Fiducia → Assess**

Fiducia should therefore be developed after Rogare and before Aestimare/Assess.

This is an important roadmap change.

### 7.1 First Fiducia Responsibility — Replay Orchestration

Fiducia's first practical capability should be orchestration of Repetere.

It should be able to define:

- which replay should run;
- when it should run;
- how many executions should occur;
- periodic or scheduled execution;
- execution history;
- the evidence accumulated from those executions.

Repetere continues to execute each individual replay.

Fiducia decides **when and how often** Repetere should be invoked.

### 7.2 Evidence and Trust

Fiducia represents a larger architectural idea than scheduling alone.

Its purpose is ultimately to:

> **Replace constant verification with calibrated trust built from evidence.**

The broader principle is:

> **Trust isn't the absence of verification. Trust is what accumulated evidence allows you to stop verifying every time.**

Repeated replay observations provide one mechanism for building that evidence.

Fiducia can therefore become the layer that understands whether previously verified behaviour remains sufficiently stable that exhaustive verification is no longer required on every execution.

### 7.3 Relationship with Assess

Fiducia comes before Assess structurally, but Assess will ultimately strengthen Fiducia.

A likely progression is:

`Repetere → evidence → Aestimare → assessment → Fiducia → calibrated trust`

Initial Fiducia development does not need the complete Assess capability. It can begin by orchestrating and retaining replay evidence.

Once Assess exists, Fiducia can consume richer behavioural assessment rather than relying only on replay completion/divergence evidence.

---

## 8. Assess — Behavioural Assessment

Aestimare/Assess remains the behavioural assessment layer.

Its governing principle remains:

> **Assess measures behaviour, not intelligence.**

Assess should evaluate whether the behaviour and answer produced by a model were sufficient for the task and circumstances.

### Checkpoint Review Before Assess

Before Assess development begins, the current Lumen checkpoint implementation must be reviewed.

Checkpoints should not be considered solely a context-window rollover/compression mechanism.

They may also represent snapshots of the model's cognitive state or working understanding at meaningful stages of a conversation.

Assess may need these snapshots to understand:

- what the model believed at different stages;
- how its understanding changed;
- what information was available before an answer;
- whether important facts or objectives were lost;
- whether the final answer was sufficiently supported by the preceding reasoning state.

Some checkpoint capability has already been implemented, but its semantics and suitability for Assess need explicit review.

### ACP `\obt` Behaviour

Before or during the approach to Assess, investigate the observed case where ACP-managed Pontis sessions caused Moderari/Lumen to emit its `\obt` help/banner text.

`\obt` should only activate when a user explicitly enters a command beginning with `\obt`.

Protocol/session setup must not be mistaken for user/model conversational content.

---

## 9. Servire Direction

Servire remains the operational surface for the complete Lumen family.

Its role should continue moving toward:

- unified component lifecycle management;
- component configuration;
- operational status;
- logs;
- Trace inspection;
- Replay control;
- Pontis visibility;
- Rogare access;
- later Fiducia scheduling and evidence;
- later Assess results and behavioural history.

Servire should not absorb the responsibilities of the components it exposes.

It is the **control plane**, not the implementation of those capabilities.

---

## 10. Development Sequence

The current intended sequence is:

### Phase 1 — Current Stack Standardisation

1. Move Trace/Vestigare to port `11438`.
2. Complete Pontis integration.
3. Rename the existing Lumen orchestrator to **Lumen Moderari**.
4. After the Moderari rename is complete, standardise Servire component navigation using **Latin component name — English responsibility**.
5. Move Moderari-specific capabilities such as Operations and Checkpoints beneath the **Moderari — Orchestrator** workspace rather than exposing them as top-level component tabs.
6. Apply the component naming convention consistently across Servire headings, status, lifecycle controls, configuration, dependency displays and operational logs.
7. Verify the complete existing stack after these changes.

### Phase 2 — Rogare

8. Develop Lumen Rogare as a separate conversational client.
9. Route Rogare through Pontis.
10. Establish ACP tool discovery/routing through Pi.
11. Validate Rogare as a tool-less client using externally supplied tools.

### Phase 3 — Fiducia

12. Remove repeated-run responsibility from Repetere specifications.
13. Develop Fiducia replay scheduling/orchestration.
14. Support replay execution counts and periodic execution.
15. Accumulate execution evidence and history.
16. Establish the initial evidence model for calibrated trust.

### Phase 4 — Assess

17. Review and formalise checkpoint semantics.
18. Resolve outstanding protocol/content classification issues such as `\obt`.
19. Develop Assess behavioural evaluation.
20. Evaluate answer sufficiency and behavioural outcomes.
21. Feed Assess evidence into Fiducia.

### Phase 5 — Integrated Reasoning Assurance

22. Build the combined Replay → Assess → Fiducia evidence cycle.
23. Surface evidence, confidence/trust and behavioural history through Servire.
24. Investigate semantic replay equivalence rather than strict textual equivalence where appropriate.
25. Develop longer-lived continuity/evidence graphs.

### Phase 6 — Public Lumen Site and Demonstration

26. Build `lumen.illuminates.one` after Assess reaches a useful level of maturity.
27. Distil the existing engineering and architectural documentation into the public story.
28. Provide an interactive demonstration, potentially using replay-driven scenarios rather than unrestricted public prompting.
29. Use a separate demonstration data store so historical runs can be managed independently.
30. Expose selected Servire capabilities while hiding operationally sensitive controls.

---

## 11. Architectural Trajectory

The emerging Lumen architecture can be viewed as a progression:

`Client → Pontis → Observation → Replay → Orchestration → Assessment → Trust`

or by component:

`Rogare → Pontis → Vestigare → Repetere → Fiducia → Aestimare`

with **Moderari** providing the core model orchestration and continuity capability and **Servire** providing the operational control plane across the system.

The important development is that Lumen is no longer being shaped as a single increasingly complex application.

It is becoming a family of narrowly responsible components whose combined purpose is to provide:

**Continuity • Understanding • Provenance • Trust**

---

## 12. Current Roadmap Principle

The immediate roadmap is deliberately architecture-first.

The priority is not to add capabilities wherever they are easiest to implement. It is to place each capability in the component that should logically own it.

That principle has already produced several important separations:

- Lumen becomes the ecosystem; Moderari becomes the orchestrator.
- Pontis owns protocol/session bridging, not conversational behaviour.
- Rogare owns the user conversation surface, not tools.
- Repetere executes one replay; it does not schedule repetitions.
- Fiducia orchestrates repeated evidence gathering and ultimately calibrated trust.
- Assess evaluates behaviour and sufficiency.
- Servire exposes and controls the system without absorbing component responsibilities.

This separation should remain an architectural invariant as the Lumen ecosystem develops.
