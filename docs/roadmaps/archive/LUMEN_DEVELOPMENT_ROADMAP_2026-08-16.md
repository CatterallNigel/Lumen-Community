# Lumen Development Roadmap

**Date:** 2026-08-16  
**Status:** Current development direction  
**Primary focus:** Fiducia and Aestimare

## 1. Purpose

This roadmap records the next development phase for Lumen following the successful establishment of the core operational stack and completion of the principal Rogare development work.

The immediate priority moves from further operational refinement toward the capabilities that extend Lumen from orchestration, observability and replay into a broader **Reasoning Assurance Platform**:

- **Fiducia** — orchestration and scheduling of repeated behavioural observations.
- **Aestimare** — assessment of the behavioural evidence produced by Lumen.

Several worthwhile Servire and operational improvements remain outstanding. They are deliberately deferred because they are not prerequisites for beginning Fiducia or Aestimare.

The architectural invariant remains:

> **Each Lumen component should have one clearly bounded responsibility.**

A further development rule follows:

> **Do not expand Repetere to solve Fiducia problems, and do not expand Fiducia to solve Aestimare problems.**

New services should nevertheless become part of the complete Lumen stack early:

> **Integrate a new Lumen service into Servire once its first useful capability, service contract and standalone UI are stable. Feature completeness is not required.**

All services must have:

> **Every Lumen component must own and expose standard Clear Logs functionality before it is considered ready for Servire integration.**

## 2. Current Stable Baseline

The operational component family now includes:

- **Servire — Service** — operational control plane.
- **Moderari — Orchestrator** — model-facing orchestration and continuity.
- **Praebere — Provider Manager** — model-provider lifecycle.
- **Pontis — Communication Bridge** — HTTP/ACP communication and session correlation.
- **Rogare — Conversational Client** — Lumen-native conversational UI.
- **Vestigare — Recorder** — behavioural recording and provenance.
- **Repetere — Replay** — individual replay execution and fork detection.

The complete stack has been exercised end-to-end with Rogare conversations, Ollama/Qwen operation through Praebere, ACP/Pi tool execution, logical session continuity, Pontis session correlation, Rogare session reattachment, heartbeat/progress backchannel, Trace recording, Replay match/fork behaviour and Servire-managed lifecycle.

This baseline is sufficient to begin building the assurance capabilities above Replay.

## 3. Priority 1 — Lumen Fiducia

### 3.1 Responsibility

Fiducia is the orchestration layer above individual Replay execution.

**Repetere remains responsible for executing one Replay Run per invocation.**

Fiducia is responsible for deciding and coordinating:

- which Prepared Replay should run;
- when it should run;
- how often it should run;
- how many Replay Runs should execute at each scheduled occurrence;
- the operational state of those scheduled executions.

Fiducia must not reproduce Repetere's replay engine.

### 3.2 Initial Fiducia Capability

```text
Prepared Replay(s)
       |
       v
Fiducia Schedule
       |
       +-- When should it run?
       +-- How often should it run?
       +-- How many Replay Runs per occurrence?
       |
       v
Repetere
       |
       v
Replay Results
```

The first implementation should support:

- discovery/viewing of Prepared Replays exposed by Repetere;
- selection of a Prepared Replay for scheduled execution;
- creation of a Fiducia schedule;
- one-time and recurring scheduling;
- configurable recurrence/frequency;
- configurable number of Replay Runs per scheduled occurrence;
- invocation of Repetere once for each individual Replay Run;
- tracking of requested, running, completed and failed runs;
- preservation of the relationship between a Fiducia schedule and resulting Replay Runs/Results;
- cancellation or disablement of future scheduled execution;
- operational logging and controlled error handling.

The scheduling vocabulary and persistence model belong to Fiducia, not Repetere.

### 3.3 Fiducia Standalone UI

A standalone Fiducia UI is part of the first stable milestone and must exist **before Servire integration**. It can initially remain deliberately simple.

The UI should provide three principal areas.

#### Available Replay Material

Display Prepared Replays made available by Repetere. The operator should be able to identify the Prepared Replay, inspect sufficient status/source information and select it for Fiducia orchestration.

Fiducia must consume Repetere's supported API rather than its internal persistence.

#### Schedule Configuration

Provide controls for:

- Prepared Replay selection;
- first/next execution time;
- one-time or recurring execution;
- recurrence/frequency;
- number of Replay Runs per scheduled occurrence;
- enabled/disabled state.

The UI must clearly distinguish schedule frequency from runs per occurrence. For example:

```text
Run every:       24 hours
Runs each time:  10
```

means Fiducia invokes Repetere ten times every 24 hours. Repetere itself still performs one Replay Run per invocation.

#### Scheduled Work

Display configured schedules and operational state, initially including:

- schedule name/identifier;
- Prepared Replay;
- enabled/disabled state;
- next scheduled execution;
- last execution;
- runs requested;
- runs completed;
- runs failed;
- current state.

Advanced analytics are not required for the first UI.

### 3.4 Fiducia Service Contract and Persistence

Before Servire integration, Fiducia should expose a stable service contract covering at least:

- health/status;
- schedule discovery and creation;
- schedule modification;
- enable/disable/cancel;
- execution state;
- Replay Run/Result references;
- component-owned log management;
- lifecycle behaviour.

Schedules must survive process restarts. Fiducia owns persistence for schedules, recurrence, enabled state, associated Prepared Replay, runs per occurrence, execution history and next-run state.

## 4. Priority 2 — Fiducia Servire Integration

Once the first Fiducia capability, API and standalone UI are stable, Fiducia should immediately become a first-class Servire-managed component. Feature completeness is not required.

Integration should include:

- component configuration;
- dependency validation;
- managed startup and shutdown;
- health/status;
- operational logging and Clear Logs integration;
- Fiducia workspace/tab;
- embedding of the Fiducia UI;
- stack lifecycle ordering;
- full-stack regression testing.

Servire operates Fiducia but must not absorb Fiducia's scheduling responsibility.

## 5. Priority 3 — Lumen Aestimare

### 5.1 Responsibility

Aestimare is the behavioural assessment service.

> **Assess measures behaviour, not intelligence.**

Repetere identifies Replay match and divergence. Vestigare records behavioural evidence. Fiducia orchestrates repeated observations. **Aestimare judges the resulting evidence.**

### 5.2 Initial Aestimare Capability

```text
Trace / Replay Evidence
          |
          v
     Aestimare
          |
          +-- What behaviour was observed?
          +-- Was it consistent or divergent?
          +-- Was the available answer/context sufficient?
          |
          v
   Assessment Result
```

The first implementation should support:

- discovery/selection of suitable Trace and Replay evidence;
- creation of an assessment;
- preservation of evidence provenance;
- assessment execution;
- structured assessment results;
- explicit association between assessment and source evidence;
- assessment status;
- operational logging;
- controlled failure states.

Aestimare should consume evidence through supported component interfaces rather than another component's internal storage.

### 5.3 Behavioural Assessment

Initial assessment should concentrate on observable behaviour, including candidates such as:

- consistency between Replay Runs;
- nature and position of divergence;
- whether divergence materially changes the outcome;
- answer sufficiency;
- context sufficiency;
- use of available capabilities/tools;
- behavioural patterns across repeated runs.

The detailed metric/scoring model belongs in Aestimare.

### 5.4 Checkpoint Use

Aestimare should be designed to consume Lumen checkpoints where they provide useful evidence of model/context understanding, including retained understanding, context state, assumptions, continuity and sufficiency.

Checkpoint-based assessment should be introduced incrementally rather than made a prerequisite for the first useful Aestimare milestone.

### 5.5 Aestimare Standalone UI

A standalone Aestimare UI is part of its first stable milestone and must exist **before Servire integration**.

The initial UI should allow the operator to:

- view/select available evidence;
- identify the originating Trace Recording or Replay Run;
- initiate an assessment where appropriate;
- inspect assessment state;
- inspect the resulting behavioural assessment.

An initial result view can show:

- source evidence;
- assessment status;
- key findings;
- consistency/divergence indication;
- sufficiency indication where supported;
- provenance/reference information;
- related Trace/Replay identifiers.

Advanced visual analytics are not required initially. The first objective is a clear, inspectable assessment result.

### 5.6 Aestimare Service Contract

Before Servire integration, Aestimare should expose a stable contract covering at least:

- health/status;
- evidence discovery/reference;
- assessment creation;
- assessment status;
- assessment result retrieval;
- component-owned log management;
- lifecycle behaviour.

Assessment data and state remain owned by Aestimare.

## 6. Priority 4 — Aestimare Servire Integration

Once the first Aestimare capability, API and standalone UI are stable, Aestimare should immediately become a first-class Servire-managed component.

Integration should include configuration, dependency validation, managed lifecycle, health/status, operational logging, Clear Logs, an Aestimare workspace/tab, embedded UI and full-stack regression testing.

As with Fiducia, integration should happen early rather than waiting for feature completeness.

## 7. Priority 5 — Fiducia and Aestimare Evolution

After both services reach their first stable capability and are integrated into Servire, development can expand their relationship.

```text
Vestigare
    |
    | behavioural evidence
    v
Repetere
    |
    | individual Replay Runs
    v
Fiducia
    |
    | repeated / scheduled observations
    v
Aestimare
    |
    | behavioural assessment
    v
Reasoning Assurance Evidence
```

This illustrates responsibility flow; it does not require every data exchange to pass physically through every component.

Potential later capabilities include:

- assessment across repeated Fiducia-controlled Replay Runs;
- longitudinal behavioural comparison;
- behavioural distribution analysis;
- recurring assurance schedules;
- comparison across models/providers;
- checkpoint-informed sufficiency assessment;
- behavioural drift detection;
- confidence/trust evidence;
- historical assessment trends;
- reports suitable for human review;
- Fiducia scheduling decisions informed by prior assessment results.

Fiducia decides **when and how often to observe**. Aestimare decides **what the evidence means**.

## 8. Deferred Operational Refinements

These remain valuable but are not current prerequisites.

### 8.1 Servire Continuous Runtime Dependency Health

Complete/verify continuous runtime dependency monitoring with aggregate states **HEALTHY**, **DEGRADED**, **STOPPED** and **FAILED**. Transient MongoDB/provider loss should degrade rather than automatically terminate the stack and recover automatically when the dependency returns.

### 8.2 Servire Health and Event Correlation

Develop richer cross-component correlation for dependency, service, Replay, Trace, Fiducia, Aestimare, checkpoint and log-management events. Servire correlates and presents these events without taking ownership of the underlying capabilities.

### 8.3 Multiline Exception Capture Verification

Explicitly verify that complete multiline Python exceptions/tracebacks are preserved and usefully presented in the Servire Operational Log.

### 8.4 Praebere / Ollama Shutdown Timing

Continue observing managed provider shutdown. Servire now initiates managed stack shutdown when Servire itself terminates, but a recent Praebere/Ollama shutdown exceeded the previous 180-second lifecycle allowance. The allowance is now 300 seconds.

If shutdown continues to approach or exceed this duration, investigate the Praebere/Ollama provider-stop path rather than continuing to increase Servire's timeout.

This remains an operational investigation unless it begins to affect normal development reliability.

## 9. Recommended Development Sequence

### Phase 1 — Fiducia Architecture and Core

1. Define Fiducia architecture and responsibility boundary.
2. Define the Fiducia/Repetere API relationship.
3. Define schedule and execution persistence.
4. Implement Prepared Replay discovery.
5. Implement one-time scheduling.
6. Implement recurring scheduling.
7. Implement configurable Replay Runs per occurrence.
8. Track scheduled execution and resulting Replay Runs.
9. Add operational logging, health and Clear Logs support.

### Phase 2 — Fiducia Standalone UI

10. Implement Prepared Replay view/selection.
11. Implement schedule configuration UI.
12. Implement scheduled-work/status view.
13. Exercise complete scheduling workflows.
14. Stabilise API/UI/service behaviour.

### Phase 3 — Fiducia Servire Integration

15. Add Fiducia to Servire configuration and dependency topology.
16. Add managed lifecycle.
17. Add health/logging integration.
18. Add Fiducia tab/workspace.
19. Embed Fiducia UI.
20. Perform full-stack regression.

### Phase 4 — Aestimare Architecture and Core

21. Define Aestimare architecture and responsibility boundary.
22. Define evidence and assessment models.
23. Define component API relationships.
24. Implement evidence discovery/reference.
25. Implement first behavioural assessment workflow.
26. Persist structured assessment results.
27. Add operational logging, health and Clear Logs support.

### Phase 5 — Aestimare Standalone UI

28. Implement evidence selection/view.
29. Implement assessment initiation/status.
30. Implement structured assessment-result view.
31. Exercise complete assessment workflows.
32. Stabilise API/UI/service behaviour.

### Phase 6 — Aestimare Servire Integration

33. Add Aestimare to Servire configuration and dependency topology.
34. Add managed lifecycle.
35. Add health/logging integration.
36. Add Aestimare tab/workspace.
37. Embed Aestimare UI.
38. Perform full-stack regression.

### Phase 7 — Assurance Evolution

39. Connect Fiducia-controlled repeated Replay evidence to Aestimare.
40. Introduce multi-run behavioural comparison.
41. Introduce checkpoint-informed assessment.
42. Develop longitudinal/drift analysis.
43. Develop assurance reporting.
44. Revisit agentic Fiducia behaviour once evidence and assessment contracts are stable.

### Phase 8 — Deferred Operational Work

45. Complete/verify Servire runtime degradation/recovery.
46. Develop richer Servire event correlation.
47. Verify multiline traceback handling.
48. Resolve Praebere/Ollama shutdown timing if still present.
49. Perform complete ecosystem regression.

## 10. First-Milestone Acceptance Criteria

### Fiducia

The first Fiducia milestone is complete when:

- Prepared Replays can be discovered through Repetere;
- a user can create a schedule from the standalone Fiducia UI;
- the schedule defines when/how often it executes;
- the schedule independently defines how many Replay Runs occur each time;
- Fiducia invokes Repetere once for each required Replay Run;
- execution state and resulting Replay references are retained;
- schedules survive restart;
- the service contract is stable;
- tests and quality checks pass;
- the standalone UI is usable;
- Fiducia is ready for immediate Servire integration.

### Aestimare

The first Aestimare milestone is complete when:

- suitable behavioural evidence can be selected/referenced;
- an assessment can be initiated from the standalone UI;
- evidence provenance remains explicit;
- Aestimare produces a structured behavioural assessment;
- the result can be inspected through the UI;
- assessment state survives restart where required;
- the service contract is stable;
- tests and quality checks pass;
- the standalone UI is usable;
- Aestimare is ready for immediate Servire integration.

## 11. Architectural Principles

> **Repetere executes one Replay Run.**

> **Fiducia decides when, how often and how many Replay Runs should occur.**

> **Aestimare determines what the resulting behavioural evidence means.**

> **Servire operates and presents the services; it does not absorb their capabilities.**

The development sequence must resist convenient but inappropriate responsibility migration.

> **Do not expand Repetere to solve Fiducia problems, and do not expand Fiducia to solve Aestimare problems.**

At the same time, new services should join the operational ecosystem early:

> **A stable core capability, stable service contract and usable standalone UI are sufficient for Servire integration. Full feature maturity is not required.**

This allows Fiducia and Aestimare to evolve inside the real Lumen stack while preserving the clear component boundaries on which Lumen's architecture depends.
