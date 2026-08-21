# Lumen Glossary and Naming Index

**Status:** Current Supporting\
**Purpose:** Provide a common vocabulary for Lumen services,
architectural concepts and historical terminology\
**Last reviewed:** 2026-08-21

------------------------------------------------------------------------

## Lumen

**Lumen** is the complete Reasoning Assurance Service ecosystem.

It is not the name of an individual service, model, agent, client or
provider.

Lumen surrounds probabilistic model execution with services that make
behaviour observable, reproducible, repeatable and assessable while
preserving provenance.

------------------------------------------------------------------------

## Service Names

  ---------------------------------------------------------------------------
  Name                    Current role                Common English
                                                      shorthand
  ----------------------- --------------------------- -----------------------
  **Moderari**            Orchestration Service       Orchestrate

  **Pontis**              Bridge Service              Bridge

  **Praebere**            Integration / Model         Provider abstraction
                          Provider Service            

  **Vestigare**           Trace Service               Trace

  **Repetere**            Replay Service              Replay

  **Fiducia**             Reasoning Assurance         Fiducia
                          automation /                
                          repeated-execution          
                          coordination                

  **Aestimare**           Assess Service              Assess

  **Rogare**              Operations/conversational   Rogare
                          UI client                   

  **Servire**             Operational Control Plane / Servire
                          Operations Service          

  **Audire**              Proposed distributed        Audire
                          operational transport       
  ---------------------------------------------------------------------------

The Latin names are the service identities. English labels describe
responsibilities and may have evolved as those responsibilities became
clearer.

------------------------------------------------------------------------

## Core Reasoning Assurance Terms

### Observe

Capture what happened during an execution.

Vestigare is the principal service associated with observation.

### Trace

The recorded evidence of an execution and its relevant
metadata/provenance.

"Trace" may also be used informally for Vestigare's responsibility, but
**Vestigare** is the service name.

### Reproduce

Attempt to execute a recorded interaction again under controlled
conditions.

### Replay

A controlled re-execution performed through Repetere.

A Replay is an execution, not an experiment containing an arbitrary
number of runs.

### Divergence

A detectable difference between executions.

Divergence is evidence. It is not automatically an error or behavioural
failure.

### Repeat

Perform multiple controlled executions so behaviour can be examined
across more than one observation.

Fiducia coordinates this repeated-execution responsibility.

### Experiment

A set of repeated controlled executions performed to investigate
behaviour under defined conditions.

### Assess

Interpret accumulated evidence to characterise model behaviour.

Aestimare owns this responsibility.

### Reasoning Assurance

The engineering process of accumulating and assessing evidence about
model behaviour and the conditions that produced it.

Reasoning Assurance does not mean proving that a model is always correct
or forcing probabilistic behaviour to become deterministic.

------------------------------------------------------------------------

## Evidence Terms

### Evidence

Recorded information capable of supporting an engineering or behavioural
claim.

### Provenance

Information describing where evidence came from and the conditions under
which it was produced.

Examples include model/provider, context, tools, orchestration
conditions, execution identifiers and relationships between Trace,
Replay, repeated runs and assessment.

### Observation

Evidence from a single execution.

A single observation can establish what happened once; it cannot by
itself characterise general behaviour.

### Behavioural Characterisation

A description of model behaviour derived from multiple
observations/experiments rather than an isolated response.

### Longitudinal Evidence

Evidence accumulated over time that permits behavioural change to be
detected or compared.

------------------------------------------------------------------------

## Behavioural Terms

### Behaviour

The observable characteristics of model execution under particular
conditions.

Aestimare assesses behaviour, not intelligence.

### Sufficiency

Whether an outcome adequately satisfies the relevant requirements or
constraints.

Sufficiency is important because many engineering/reasoning tasks have
multiple acceptable solutions rather than one uniquely correct answer.

### Deterministic Failure

A failure mode in which the same conditions reliably produce the same
incorrect or failed result.

### Probabilistic Divergence

Variation between executions that can arise even under apparently
equivalent conditions.

Lumen records and assesses this variation rather than treating all
variation as failure.

### Context Drift / Contextual Behaviour

Behavioural change caused or influenced by the accumulated execution
context.

In long sessions, the model may retain explicit context that the human
user is no longer actively considering. Earlier ambiguity or
contradiction can therefore affect later responses.

------------------------------------------------------------------------

## Architectural Terms

### Bounded Responsibility

The rule that each Lumen service has one primary architectural job and
should not absorb another service's responsibility merely for
implementation convenience.

### Conversational Execution Plane

The path through which user/model interactions are processed.

### Evidence Plane

The progression through capture, Replay, repeated execution and
assessment.

### Operational Control Plane

The functions used to start, stop, validate, observe and operate the
Lumen services. Servire owns this plane.

### Model Provider

An external system that supplies model execution.

The provider is external to Lumen. Praebere abstracts provider-specific
lifecycle/readiness behaviour.

### Capability Provider / Tool Provider

An external provider that supplies a tool, semantic capability or other
callable function.

External providers return results or semantics; they do not own Lumen's
orchestration prompts.

### Client

A user-facing or machine-facing consumer of Lumen. A client is not
itself the Lumen assurance architecture.

------------------------------------------------------------------------

## Service-Specific Terms

### Moderari

The service responsible for controlled orchestration at the model
boundary, including Lumen-owned intervention/instruction behaviour.

### Pontis

The bridge responsible for client/session correlation and protocol
boundaries between clients, Lumen and supported external capability
paths.

### Praebere

The Lumen-side abstraction for external model-provider lifecycle,
readiness and model availability.

### Vestigare

The service responsible for Trace/evidence capture.

### Repetere

The service responsible for controlled Replay and recording divergence.

### Fiducia

The service responsible for coordinating repeated controlled Replay
activity.

### Aestimare

The service responsible for behavioural assessment of accumulated
evidence.

Aestimare may internally coordinate specialist assessment engines while
retaining one external service contract.

### Peritus / Periti

Proposed terminology for Aestimare's specialist internal assessment
engines.

A **Peritus** independently assesses one specialised dimension of
behaviour. **Periti** is plural.

### Periti Assessores

Collective term proposed for Aestimare's specialist internal assessors.

This terminology describes an internal Aestimare design direction; it
does not create additional top-level Lumen services.

### Rogare

The human-facing conversational/operational client used to interact with
the Lumen stack.

### Servire

The operational control plane responsible for service lifecycle,
dependency validation, operational state and logs/control.

### Audire

Proposed distributed operational transport intended to carry service
output, commands and acknowledgements between distributed services and
Servire.

Audire changes transport; it does not replace Servire's authority.

------------------------------------------------------------------------

## Historical / Evolving Terms

### Lumen Console

Historical concept for a more unified Lumen console/UI responsibility.

The architecture subsequently separated responsibilities, principally
into Servire's operational control plane and Rogare's user-facing
interaction role.

Documents named `LUMEN-CONSOLE*` are historical records.

### UI Vision

Historical design material created before the current Servire/Rogare
architecture solidified.

`VISION-UI.md` and `VISION-UI-v2.md` are historical.

### Architecture Appendices

The original Lumen architecture was extended through two appendices
while the architecture was still evolving.

Those three documents are retained as historical provenance. The current
canonical architecture is `docs/architecture/core/ARCHITECTURE.md`.

------------------------------------------------------------------------

## Documentation Status Terms

### Canonical

The authoritative current source for a subject.

### Current

Accurate and actively relevant now, but not necessarily the sole
authority.

### Current Supporting

Accurate supporting documentation that complements a canonical source.

### Historical

A preserved record of Lumen at an earlier point in its development. It
should not be interpreted as current architecture.

### Superseded

A document explicitly replaced by a newer source serving essentially the
same documentary purpose.

### Proposal

A design or change under consideration but not established as current
architecture.

### Research

Exploratory material intended to inform Lumen rather than define
implemented architecture.

### Evidence

Material retained to substantiate an engineering, behavioural or
milestone claim.

### Publication

Material prepared for external/public communication.

For the formal lifecycle rules, see
`docs/standards/DOCUMENT_STATUS_AND_LIFECYCLE.md`.

------------------------------------------------------------------------

## Naming Principle

When terminology conflicts, prefer:

1.  the current canonical architecture;
2.  the current ecosystem topology/service matrix;
3.  the current service's own documentation;
4.  this glossary;
5.  historical documents only for the period they describe.

Historical names should not be silently rewritten out of historical
documents. Their value is partly that they preserve how Lumen's
architecture and language evolved.
