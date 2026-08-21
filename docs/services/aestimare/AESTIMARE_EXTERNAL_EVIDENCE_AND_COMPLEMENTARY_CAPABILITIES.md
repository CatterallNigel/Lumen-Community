# Aestimare --- External Evidence and Complementary Capability Integration

**Status:** Architectural discussion note\
**Component:** Lumen Aestimare (Assess)\
**Date:** 2026-08-20

## 1. Purpose

This note records an architectural principle for future development of
**Lumen Aestimare**:

> **Lumen should be able to benefit from specialised external
> capabilities without absorbing them, conflating their responsibilities
> with Lumen's own, or surrendering responsibility for what their
> evidence means.**

The objective is **integration without conflation**. External systems
retain their intellectual, architectural and operational boundaries.
Lumen interacts with them through explicit interfaces, preserves
provenance, and determines how their contributions affect an assessment.

## 2. Foundational Observation

> **A single execution is an observation. Repeated controlled executions
> become an experiment. Multiple experiments begin to characterise
> behaviour. Longitudinal evidence allows us to detect behavioural
> change.**

This progression expands Aestimare beyond comparing two outputs and
assigning a score:

**Observation → Experiment → Behavioural Characterisation → Longitudinal
Change**

The evidence needed to do this need not originate exclusively inside
Lumen.

## 3. Preserve Boundaries

The goal is not to combine external research or systems into a single
Lumen implementation.

Each capability remains responsible for what it uniquely understands:

-   Lumen remains responsible for Reasoning Assurance.
-   An external semantic capability remains responsible for the
    semantics it produces.
-   An external governance or behavioural-boundary capability remains
    responsible for its own model and outputs.
-   Aestimare remains responsible for deciding what externally supplied
    evidence means in the context of an assessment.

Independent approaches may use different theoretical foundations,
measure different phenomena, disagree, evolve independently, and expose
different confidence models. Lumen should not erase those differences.
They may themselves become useful evidence.

## 4. External Capabilities as Evidence Providers

A core architectural principle is:

> **Aestimare should be extensible in what evidence it can consider
> without surrendering responsibility for what that evidence means.**

External capabilities might contribute semantic interpretation,
behavioural classification, governance/boundary evaluation, contextual
analysis, domain-specific assessment, cognitive or behavioural
indicators, or independent confidence measures.

These are **inputs to assessment**, not automatically assessment
conclusions.

An external system might report a boundary violation. Aestimare must
still determine which boundary was involved, under what conditions,
whether it was relevant to the task, whether it is reproducible, whether
other evidence agrees, and whether the difference is material.

> **The external capability informs the assessment. It does not silently
> become the assessment.**

## 5. Provenance Is Mandatory

Every external contribution must retain provenance. Aestimare should
distinguish, for example:

-   observation produced by Vestigare;
-   replay evidence produced by Repetere;
-   execution/context evidence preserved through Moderari;
-   semantic evidence produced by an external capability;
-   governance/boundary evidence produced by another external
    capability;
-   assessment produced by Aestimare.

Where applicable, provenance should retain provider/capability identity
and version, configuration, supplied inputs, returned output, timestamp,
model/provider information, confidence or uncertainty, and
failures/incomplete results.

Externally generated conclusions must never become unattributed facts
inside the assurance record.

## 6. Independence Can Be Valuable

Agreement between independent methods can strengthen evidence.
Disagreement can be equally informative.

If Lumen classifies an event as acceptable variation, an external
boundary evaluator reports a possible violation, and a semantic
evaluator reports no material semantic change, Aestimare should not
automatically reconcile those results.

The disagreement may reveal different definitions, sensitivity,
incomplete assessment, overly restrictive rules, or genuine ambiguity.

> **Reasoning Assurance should value independent evidence, not merely
> consensus.**

## 7. Praebere as an Integration Boundary

Where appropriate, external AI/model-facing capabilities should be
integrated through **Lumen Praebere**, preserving its existing
principle:

> **Lumen components know Lumen interfaces; provider adapters know
> external providers.**

Conceptually:

``` text
Aestimare
    |
Lumen-defined capability request
    |
Praebere
    |
    +--> Provider / Model
    +--> External Semantic Capability
    +--> External Governance Capability
    +--> Other Specialised Capability
```

Not every external system necessarily belongs behind Praebere. The exact
integration boundary depends on its nature. The important requirement is
that Aestimare should not become tightly coupled to a third party's API,
ontology or implementation.

## 8. Current Examples

### Monica's research

Monica's glyph/semantic work is a candidate independent third-party
capability. Lumen should not reproduce it internally. It could expose
semantic evidence through a defined interface, allowing Aestimare to
consider that evidence alongside Lumen's own observations.

The boundary remains:

**External semantic capability → semantic evidence**\
**Aestimare → meaning of that evidence within the assessment**

### Daniel's work

Recent discussion with Daniel suggests a potentially complementary
direction involving governance, behavioural/cognitive boundaries and the
environment surrounding a model.

His material has not yet been fully reviewed, so no architectural
conclusion should currently be drawn.

A hypothesis worth investigating is:

**External governance/boundary capability → conditions, constraints or
behavioural evidence**\
**Lumen → observation, reproduction, assessment and longitudinal
assurance**

Daniel's phrase **"different links in the same chain"** should remain a
hypothesis until his system and responsibilities are properly
understood. His capability might eventually integrate through Praebere
or another appropriate Lumen boundary.

## 9. Complementary Systems, Not Dependencies

Aestimare should remain useful without any particular optional external
capability.

Third-party systems should normally be complementary evidence providers
rather than mandatory dependencies. This keeps Lumen provider-agnostic,
allows external research to evolve independently, permits substitution
and comparison, and avoids creating new single points of failure.

## 10. Relationship to Fiducia

Aestimare evaluates evidence. Fiducia eventually accumulates evidence
into justified confidence.

A future assurance record may therefore need to establish:

-   What was observed?
-   Who observed it?
-   How was it assessed?
-   Which external capabilities contributed?
-   Were those capabilities independent?
-   Did their conclusions agree?
-   Was the behaviour reproducible?
-   Has the same pattern appeared historically?
-   How reliable has each evidence source itself proven to be?

This creates the possibility that Fiducia can eventually reason not only
about model behaviour but also about the **quality and history of the
evidence used to assess that behaviour**.

> **Trust in the system should not depend blindly on trust in an
> evaluator. Evidence itself requires provenance.**

## 11. Implications for Aestimare Architecture

Before implementation, Aestimare should explicitly support:

1.  **Typed evidence** --- do not immediately reduce every contribution
    to a common score.
2.  **Provenance** --- retain origin, version, conditions and
    confidence.
3.  **Separation of evidence production and assessment** --- providers
    produce evidence; Aestimare determines meaning.
4.  **Multiple evidence providers** --- allow complementary or competing
    analyses.
5.  **Representable disagreement** --- conflicting evidence must not be
    silently normalised away.
6.  **Replaceable external capabilities** --- prefer Lumen contracts to
    provider-specific coupling.
7.  **Longitudinal evidence** --- assessment results must remain useful
    to later experiments and Fiducia.
8.  **Optionality** --- core Aestimare behaviour must not depend on one
    external research project, provider or model.

## 12. Architectural Direction

Conceptually:

``` text
                       External Capabilities
                    /          |           \
             Semantics     Governance      Other
                  \            |            /
                   \           v           /
                    +------ Praebere ------+
                              |
                              v

Moderari ---> Vestigare ---> Repetere ---> Aestimare ---> Fiducia
     |             |             |             |             |
 conditions    observation   experiment    assessment     assurance
     |             |             |             |             |
     +-------------+-------------+-------------+-------------+
                       provenance/evidence
```

This is conceptual, not a final topology.

> **External systems contribute capability and evidence. Lumen preserves
> provenance and retains responsibility for Reasoning Assurance.**

## 13. Research Principle

The behavioural-science work provides one principle:

> **Borrow the methodology, not the ontology.**

The integration architecture extends it:

> **Borrow the capability, not the responsibility.**

Lumen does not need to own every method capable of producing useful
evidence. It does need to know where the evidence came from, under what
conditions it was produced, what it claims, how reliable it has proven
to be, whether it agrees with other evidence, and what it means for the
current assessment.

## 14. Pre-Development Questions

Before Aestimare implementation begins, the architecture discussion
should answer:

1.  What constitutes an Aestimare evidence object?
2.  Which evidence types belong in the initial schema?
3.  How is provenance represented?
4.  How are external confidence and uncertainty represented without
    treating them as truth?
5.  How does Aestimare represent disagreement?
6.  What belongs behind Praebere and what requires another integration
    boundary?
7.  How are external capabilities discovered and invoked?
8.  Which evidence must be preserved for Replay?
9.  Which assessment outputs must be retained for longitudinal
    comparison?
10. What information must eventually pass to Fiducia?
11. How do we prevent an external evaluator becoming an unexamined
    source of truth?
12. How can Aestimare compare evidence generated by different versions
    of the same external capability?

These questions should be resolved before implementation constrains the
architecture.

## 15. Summary

Aestimare is evolving from the idea of an output/divergence evaluator
toward an experimental assessment layer capable of contributing to
behavioural characterisation.

That does not require Lumen to implement every relevant discipline.

> **Retain the boundaries. Preserve the provenance. Integrate the
> evidence. Assess what it means.**

Monica's research, Daniel's work, behavioural psychology and future
specialised systems may illuminate different parts of the problem. Their
value to Lumen lies precisely in remaining distinguishable.

If independently developed capabilities can contribute evidence through
clear interfaces, Aestimare can bring that evidence together without
erasing its origins or surrendering its responsibility.

This creates a broader foundation for Reasoning Assurance:

> **Not one system claiming to understand everything, but an
> architecture capable of learning from multiple independent ways of
> observing and evaluating behaviour.**
