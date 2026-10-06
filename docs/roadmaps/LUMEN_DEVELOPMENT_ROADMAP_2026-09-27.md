# Lumen Development Roadmap

**Date:** 2026-09-27\
**Status:** Current development direction\
**Supersedes:** `LUMEN_DEVELOPMENT_ROADMAP_2026-08-16.md`\
**Primary focus:** Session architecture, Moderari, concurrent evidence
capture, provider independence and the foundation for Aestimare

------------------------------------------------------------------------

## 1. Purpose

This roadmap records the next development phase for Lumen following
completion of the principal M0.1 engineering work and preparation of the
M0.1 Research Distribution.

The August 2026 roadmap correctly established the responsibility
boundaries between Repetere, Fiducia, Aestimare, Vestigare and Servire.
Those boundaries remain important.

However, implementation of M0.1 and the subsequent investigation of
additional model providers exposed a more fundamental prerequisite that
was not sufficiently visible in August:

> **Lumen must move from managing an interaction to managing multiple
> independent, observable execution sessions.**

Before expanding provider support, client support or beginning
substantial Aestimare implementation, Lumen should make session
ownership explicit across orchestration, model selection and evidence
capture.

The immediate development direction is therefore:

``` text
Moderari responsibility and compaction
                |
                v
Session -> Provider -> Model ownership
                |
                v
Concurrent Vestigare evidence recording
                |
                v
Multiple independent sessions
                |
                v
Provider diversity and client interoperability
                |
                v
Aestimare / broader Reasoning Assurance
```

This is not a rejection of the August roadmap. It is the architectural
foundation that subsequent M0.1 engineering showed must exist before the
assurance capabilities described there can be developed cleanly.

------------------------------------------------------------------------

## 2. Current Stable Baseline --- M0.1

The M0.1 engineering baseline includes the principal Lumen service
family:

-   **Servire** --- operational control plane.
-   **Moderari** --- model-facing orchestration and continuity.
-   **Praebere** --- model-provider discovery, selection and lifecycle.
-   **Pontis** --- communication bridge, execution-session boundary and
    correlation.
-   **Rogare** --- Lumen-native conversational client.
-   **Vestigare** --- behavioural evidence and provenance recording.
-   **Repetere** --- controlled Replay experimentation.
-   **Fiducia** --- scheduling and repeated Replay orchestration.
-   **Nuntius** --- messaging and service communication support.

Aestimare remains outside the M0.1 Research Distribution and represents
a principal future assurance capability.

M0.1 established and exercised end-to-end Lumen execution,
provider/model discovery and selection, execution-session lifecycle,
model reservation and release, conversational continuity, Trace
recording, Replay Experiment/Run persistence, replay-created child Trace
evidence, match/divergence recording, scheduled Replay through Fiducia,
Servire operational control, runtime provider health/recovery, Docker
distribution, and research-distribution authorization/licensing.

This baseline should remain a stable research reference. Post-M0.1
architectural work should not be folded back into M0.1 merely because it
improves the design.

------------------------------------------------------------------------

## 3. Architectural Invariants

The August roadmap established:

> **Each Lumen component should have one clearly bounded
> responsibility.**

That remains a primary architectural rule.

The following responsibility boundaries remain valid:

> **Repetere executes one Replay Run.**

> **Fiducia decides when, how often and how many Replay Runs should
> occur.**

> **Vestigare records evidence.**

> **Aestimare determines what the evidence means.**

> **Servire operates and presents the services; it does not absorb their
> capabilities.**

Two further principles now become explicit:

> **Session identity is the primary execution boundary through which
> context, provider, model, activity and evidence are correlated.**

> **Provider independence means supporting different provider semantics
> through explicit bindings/adapters, not forcing every provider into
> one assumed protocol.**

------------------------------------------------------------------------

# 4. Priority 1 --- Redefine Moderari in the Current Lumen Architecture

## 4.1 Why Moderari Comes First

Moderari predates much of the present Lumen service architecture. Its
responsibilities developed when context management, continuity, model
interaction and orchestration were more tightly coupled than they are
today.

Lumen now has explicit services for provider management, session
correlation, evidence recording, Replay, scheduling, operations and
messaging.

Moderari should therefore be reviewed before further feature or UI work.
The objective is not simply to tidy the existing service. The objective
is to define what Moderari should own **now**.

## 4.2 Proposed Responsibility

Moderari should remain responsible for construction and management of
the model-facing interaction context.

``` text
Session state
     +
Available context
     +
Orchestration policy
     |
     v
 Moderari
     |
     v
Model-facing interaction
```

Moderari should not become authoritative for provider lifecycle, global
model selection, Lumen session identity, evidence interpretation or
Stack operations.

## 4.3 Required Moderari Review

The first engineering activity should define:

-   inputs and outputs;
-   state owned by Moderari;
-   state owned by the execution Session;
-   provider/model information Moderari needs but does not own;
-   context construction responsibilities;
-   compaction responsibilities;
-   evidence emitted when context is transformed;
-   lifecycle behaviour;
-   API/service contract;
-   responsibilities that should be removed from the current
    implementation;
-   UI responsibilities that remain appropriate.

Substantial Moderari UI restructuring should follow this responsibility
review rather than precede it.

------------------------------------------------------------------------

# 5. Priority 2 --- Make Context Compaction Observable

Compaction originated as a practical response to bounded model context.
In a Reasoning Assurance system, reducing or transforming context may
change subsequent model behaviour.

Therefore:

> **Compaction is an observable transformation of session context.**

Lumen should preserve sufficient evidence to determine when and why
compaction occurred, which session was affected, relevant provenance
before the transformation, what was retained/removed/transformed, the
replacement state, policy/version used and the subsequent model
interaction.

This does not require unrestricted permanent storage of all context. The
requirement is that the transformation can be understood sufficiently to
investigate whether it materially influenced later behaviour.

The Moderari redesign must determine whether compaction is initiated by
Moderari, requested through session policy, triggered by provider/model
capability, or some combination. Regardless of trigger, evidence about
the transformation should remain associated with the execution session.

------------------------------------------------------------------------

# 6. Priority 3 --- Session → Provider → Model Ownership

## 6.1 Current Limitation

M0.1 model selection and reservation were intentionally designed around
the immediate requirements of the research distribution. That
architecture becomes restrictive when multiple sessions must execute
independently.

The target relationship is:

``` text
Session
   |
   v
Provider Binding
   |
   v
Model Binding
```

rather than:

``` text
Lumen Runtime
   |
   v
Selected Model
```

## 6.2 Session Binding

A Lumen execution session should be able to resolve a binding
conceptually containing:

``` text
session_id
provider_id
model_id
provider adapter / protocol
provider-specific execution metadata
```

The exact persistence and API schema remains to be designed.

> **Provider and model selection become properties of an execution
> session rather than global properties of Lumen.**

## 6.3 Target Behaviour

``` text
Session A -> Ollama  -> qwen2.5-coder:14b
Session B -> Ollama  -> gemma3:4b
Session C -> Colibri -> qwen36-35b-a3b
```

These should eventually operate without one session redefining another
session's provider/model state.

## 6.4 Ownership Questions

The design must establish which service creates the authoritative
Session, where provider/model bindings are persisted, Praebere's
session-scoped lifecycle responsibilities, Pontis correlation behaviour,
how Moderari receives session state, how Rogare/Repetere/Fiducia request
or inherit bindings, terminal cleanup, provider failure isolation and
evidence attribution.

------------------------------------------------------------------------

# 7. Priority 4 --- Concurrent Vestigare Recording

Single-interaction execution naturally permits implicit concepts such as
`current session`, `current trace`, `active execution` and
`selected model`. Those concepts cannot remain process-global in a
concurrent Lumen.

Vestigare must be capable of recording evidence from multiple execution
streams at the same time.

Its responsibility remains evidence recording, not deciding whether
behaviour was correct or significant.

> **Vestigare records evidence. Aestimare interprets evidence.**

Concurrent recording requires explicit correlation metadata. A future
event representation may need concepts such as:

``` text
event_id
timestamp
session_id
trace_id
parent_trace_id
correlation_id
source_service
provider_id
model_id
event_type
evidence payload
```

This is illustrative rather than a final schema.

------------------------------------------------------------------------

# 8. Priority 5 --- Identity and Correlation Across Execution

Future Lumen observation may involve clients, Lumen sessions, providers,
models, tools, agents, sub-agents, Replay Runs, Trace recordings and
external execution systems.

They do not need one universal identifier. Lumen needs to preserve
sufficient relationships between their identifiers.

> **Recording does not require complete understanding at recording
> time.**

Vestigare can preserve identifiers, timestamps, parent/child
relationships, service origin and correlation evidence. Aestimare can
later reconstruct and interpret the relationship.

``` text
Execution A -----\
Execution B ------> Vestigare ----> Evidence ----> Aestimare
Execution C -----/                           correlation / interpretation
```

This becomes particularly important for agentic and swarm-style systems.

------------------------------------------------------------------------

# 9. Priority 6 --- Prove Multiple Independent Sessions

Before adding significant provider or client complexity, Lumen should
demonstrate:

``` text
Session A -> Ollama -> Model A
Session B -> Ollama -> Model B
```

executing concurrently.

Acceptance should demonstrate independent session identity, Moderari
context, model binding, provider lifecycle, concurrent Vestigare
recording, no cross-session contamination, no model-selection
interference, correct terminal cleanup and independently inspectable
evidence.

Only after this works reliably should provider diversity become the next
variable.

------------------------------------------------------------------------

# 10. Priority 7 --- Provider Independence

Investigation of Colibri demonstrated a practical reason to support
providers beyond Ollama and exposed a more important architectural
point:

> **Provider abstraction must not mean assuming that every provider
> behaves like an OpenAI-compatible endpoint.**

Providers may differ in transport, lifecycle, discovery,
loading/unloading, streaming, context handling, tool support, health
semantics, metadata and failure behaviour.

The target is:

``` text
Session
   |
   v
Provider Binding
   |
   v
Provider Adapter / Protocol
   |
   v
Model
```

OpenAI-compatible transport remains useful where naturally supported; it
should not become a hidden architectural requirement.

Once session-bound model ownership and concurrent recording are stable,
Ollama + Colibri becomes the first useful multi-provider validation.

------------------------------------------------------------------------

# 11. Priority 8 --- Client Interoperability

Additional clients should follow the session/provider foundation.
LibreChat remains a useful interoperability candidate.

The principal question becomes:

> **How does this client establish, use and terminate a Lumen execution
> session?**

Client diversity and provider diversity should remain separate concerns
joined through the Lumen session boundary.

------------------------------------------------------------------------

# 12. Priority 9 --- Aestimare

Aestimare remains a principal objective. The sequencing change improves
the evidence foundation on which it will operate.

Aestimare should ultimately be capable of analysing evidence involving
individual Trace recordings, Replay Runs, Fiducia observations,
model/provider differences, context/compaction events, session
lifecycle, tools, agents, parent/child relationships, concurrent
execution paths and cross-session patterns.

A central Reasoning Assurance distinction is:

``` text
Was the final answer acceptable?
```

is not equivalent to:

``` text
Was the execution that produced the answer acceptable?
```

A useful final answer does not establish that the intended
model/provider/context was used, tools and external resources were used
appropriately, agent hand-offs occurred as expected, or intermediate
execution remained within intended boundaries.

Aestimare is therefore more than answer scoring. It is the service
through which recorded execution evidence can become **Reasoning
Assurance evidence**.

------------------------------------------------------------------------

# 13. Priority 10 --- Fiducia Evolution

Fiducia's existing responsibility remains:

> **Fiducia decides when, how often and how many Replay Runs should
> occur.**

Its M0.1 capability should remain stable while the underlying session
architecture evolves.

Later Fiducia development can exploit session-bound provider/model
selection, repeated experiments across models/providers, assurance
schedules, Aestimare-informed observations, longitudinal observation and
drift detection.

------------------------------------------------------------------------

# 14. Deferred Work

The following should not drive the immediate redesign:

-   many model providers;
-   many clients;
-   large-scale Aestimare implementation before evidence contracts
    stabilise;
-   agent/swarm-specific architecture before general correlation works;
-   broad UI redesign before service boundaries are clear;
-   forcing providers through one transport abstraction;
-   changing the frozen M0.1 baseline to include post-M0.1 architecture.

------------------------------------------------------------------------

# 15. Recommended Development Sequence

## Phase 1 --- Moderari Architecture

1.  Document current Moderari responsibilities.
2.  Define the intended modern Moderari boundary.
3.  Identify responsibilities that now belong elsewhere.
4.  Define session-scoped Moderari state.
5.  Define Moderari inputs/outputs and context construction.
6.  Define the service contract.
7.  Reconcile the UI with the resulting responsibility.

## Phase 2 --- Compaction

8.  Define compaction triggers and policy.
9.  Define input/output state.
10. Define evidence emitted for context transformation.
11. Associate compaction evidence with Session/Trace.
12. Exercise behavioural continuity across compaction.
13. Determine evidence required for later Aestimare analysis.

## Phase 3 --- Session / Provider / Model Architecture

14. Define authoritative Session ownership.
15. Define provider-binding ownership.
16. Define model-binding ownership.
17. Define Praebere session-scoped lifecycle.
18. Define Pontis correlation.
19. Define Moderari consumption of bindings.
20. Define persistence, cleanup and failure/recovery.

## Phase 4 --- Per-Session Model Selection

21. Implement session-bound model selection.
22. Remove inappropriate runtime-global model assumptions.
23. Exercise two independent sessions against Ollama.
24. Exercise two different models concurrently.
25. Verify lifecycle isolation and cleanup.

## Phase 5 --- Concurrent Vestigare

26. Identify single-current-session/Trace assumptions.
27. Define concurrent event/recording contract.
28. Implement explicit session/trace correlation.
29. Support simultaneous recordings.
30. Verify interleaved persistence and independent terminal states.
31. Verify Replay parent/child relationships.

## Phase 6 --- Integrated Concurrent Execution

32. Execute multiple live sessions concurrently.
33. Verify Moderari context isolation.
34. Verify provider/model isolation.
35. Verify Vestigare evidence separation.
36. Verify no cross-session state leakage.
37. Verify failure isolation.
38. Perform lifecycle regression.

## Phase 7 --- Provider Abstraction

39. Define provider capability contract.
40. Define adapter/protocol boundary.
41. Reconcile Ollama against the contract.
42. Implement Colibri support.
43. Exercise Ollama and Colibri independently and concurrently.
44. Record provider provenance.
45. Verify provider failure isolation.

## Phase 8 --- Client Interoperability

46. Define client/session establishment contract.
47. Reconcile Rogare.
48. Exercise Pi/ACP.
49. Integrate/test LibreChat.
50. Verify client identity/session correlation.
51. Prevent client protocol leaking into provider ownership.

## Phase 9 --- Aestimare Foundation

52. Define evidence-consumption contract.
53. Define assessment/provenance models.
54. Consume Vestigare evidence through supported interfaces.
55. Implement first structured assessment.
56. Incorporate Replay and session/provider/model provenance.
57. Introduce context/compaction evidence where useful.
58. Provide inspectable assessment results.

## Phase 10 --- Assurance Evolution

59. Connect Fiducia observations to Aestimare.
60. Introduce multi-run comparison.
61. Compare behaviour across models/providers.
62. Introduce checkpoint/context-informed assessment.
63. Develop longitudinal/drift analysis.
64. Develop cross-session correlation.
65. Investigate agent/swarm evidence reconstruction.
66. Develop human-reviewable Reasoning Assurance reporting.

------------------------------------------------------------------------

# 16. Near-Term Acceptance Milestone

The first major post-M0.1 milestone is complete when:

``` text
Session A -> Model A
Session B -> Model B
```

can run concurrently through one Lumen installation while each session
retains independent context/model binding, Moderari uses the correct
session state, Vestigare records both streams concurrently, evidence
remains unambiguously attributable, termination releases only
appropriate resources, failure in one execution does not corrupt
another, and evidence can be inspected independently.

This milestone deliberately does not require Colibri, LibreChat or
Aestimare.

It proves their architectural foundation.

------------------------------------------------------------------------

# 17. Subsequent Acceptance Milestone

The next milestone extends the architecture to:

``` text
Session A -> Ollama  -> Model A
Session B -> Colibri -> Model B
```

with correct provider/model binding, adapter selection, lifecycle
isolation, evidence provenance, concurrent recording and failure
isolation.

This demonstrates genuine provider independence rather than provider
substitution.

------------------------------------------------------------------------

# 18. Architectural Direction

The August roadmap's responsibility flow remains useful:

``` text
Vestigare
    |
Repetere
    |
Fiducia
    |
Aestimare
    |
Reasoning Assurance Evidence
```

The September roadmap adds the execution foundation beneath it:

``` text
                    Lumen Session
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
    Moderari      Provider / Model    Vestigare
        |                |                |
        +-------- Execution -------------+
                         |
                         v
                      Evidence
                         |
             +-----------+-----------+
             |                       |
             v                       v
         Repetere                  Fiducia
             \                       /
              \                     /
               +------ Evidence ----+
                         |
                         v
                     Aestimare
                         |
                         v
              Reasoning Assurance
```

This describes responsibility/evidence relationships, not a requirement
that all data physically traverse every component.

------------------------------------------------------------------------

# 19. Working Definition of the Next Lumen Phase

> **Make execution-session identity the stable boundary across
> orchestration, provider/model ownership and evidence capture.**

From that foundation Lumen can support independent simultaneous
sessions, independent models, multiple providers, multiple clients,
concurrent research experiments, richer Replay/Fiducia experiments,
agent/swarm observation, cross-session analysis, Aestimare and stronger
Reasoning Assurance.

The current progression is:

``` text
Make execution identity explicit
        |
        v
Make state session-scoped
        |
        v
Make evidence concurrent and attributable
        |
        v
Expand providers and clients
        |
        v
Assess what the evidence means
```

That is the current Lumen development direction as of 27 September 2026.
