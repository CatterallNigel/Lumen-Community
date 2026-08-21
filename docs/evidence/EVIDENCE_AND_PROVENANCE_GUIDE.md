# Lumen Evidence and Provenance Guide

**Status:** Current Supporting\
**Purpose:** Define what repository evidence is for, what belongs under
`docs/evidence/`, and how evidence should remain attributable\
**Last reviewed:** 2026-08-21

------------------------------------------------------------------------

## Purpose

Reasoning Assurance depends on evidence, but the Lumen Community
repository is not intended to become an indiscriminate dump of runtime
data.

This guide defines the role of **repository evidence** and distinguishes
it from the operational evidence stored and correlated by Lumen
services.

The central rule is:

> Evidence retained in the repository should support a specific
> engineering, behavioural, milestone or publication claim and should
> preserve enough provenance for another reader to understand what it
> demonstrates.

------------------------------------------------------------------------

## Repository Structure

The present evidence area is:

``` text
docs/evidence/
  screenshots/
  test-results/
```

Use these folders for durable evidence that is useful to the Community
repository.

Do not place general architecture, design proposals, research notes or
roadmaps under `evidence/`.

------------------------------------------------------------------------

## What Belongs in `test-results/`

Retain test output when it materially demonstrates a development state
or engineering claim, for example:

-   a milestone test run;
-   regression results associated with a significant change;
-   coverage evidence cited by a release or Engineering Diary entry;
-   integration or end-to-end evidence showing multiple Lumen services
    operating together;
-   results needed to substantiate a published implementation claim.

Routine local test output does not need permanent repository retention
simply because it exists.

A useful test-result artefact should make clear, where available:

-   service/component;
-   date;
-   version or milestone;
-   test command or test class;
-   result;
-   relevant environment/model/provider;
-   related Diary, version-history or architecture entry.

------------------------------------------------------------------------

## What Belongs in `screenshots/`

Screenshots should be retained when the visual state itself is useful
evidence, for example:

-   a significant operational milestone;
-   a UI state discussed in engineering documentation;
-   proof of an end-to-end flow;
-   a behavioural result where the rendered interaction matters;
-   a publication or historical record that refers to the image.

Screenshots should not be treated as the preferred representation of
text that can be preserved more reliably as structured or textual
evidence.

Where practical, the surrounding document should explain what the
screenshot demonstrates.

------------------------------------------------------------------------

## What Does Not Belong in `evidence/`

Do not use `docs/evidence/` for:

-   canonical architecture;
-   service design documents;
-   proposals;
-   research papers or research notes;
-   roadmaps;
-   ordinary version history;
-   arbitrary debugging output with no continuing evidential value;
-   complete operational Trace/Replay stores merely for archival
    convenience.

Those materials already have more appropriate locations in the
repository or in Lumen's operational data stores.

------------------------------------------------------------------------

## Operational Evidence Versus Repository Evidence

Lumen's runtime evidence model is broader than the Community repository.

Operationally:

``` text
Execution
   |
Vestigare        -> observation / trace evidence
   |
Repetere         -> replay and divergence evidence
   |
Fiducia          -> repeated controlled executions
   |
Aestimare        -> behavioural assessment
```

The repository may contain selected evidence *about* these activities,
but it is not the authoritative runtime store for every execution.

A repository artefact should therefore not be mistaken for the complete
evidence graph available to Lumen itself.

------------------------------------------------------------------------

## Provenance Requirements

Evidence is useful only when its origin can be understood.

Where applicable, retained evidence should be correlatable to:

-   date/time;
-   Lumen service/version;
-   model and provider;
-   execution or Replay identifier;
-   relevant Trace;
-   experiment/repeated-run grouping;
-   test or task being performed;
-   configuration or orchestration conditions;
-   related Engineering Diary entry;
-   related milestone/version-history document;
-   assessment derived from the evidence.

Not every artefact will contain every field. Preserve the fields
necessary to understand the claim the artefact supports.

------------------------------------------------------------------------

## Evidence Chain

The preferred relationship is:

``` text
Engineering / behavioural claim
          |
          v
Documented milestone, decision or experiment
          |
          v
Identifiable evidence
          |
          v
Execution conditions / provenance
```

The direction should also work in reverse: when encountering an evidence
artefact, a reader should be able to determine why it was retained.

------------------------------------------------------------------------

## Engineering Diary Relationship

The Engineering Diary is a narrative engineering record, not an evidence
store.

When a Diary entry makes a significant claim that is supported by
retained test output or screenshots, the entry or an associated
milestone document should identify the evidence where practical.

Conversely, evidence should not be retained solely because it might
someday be useful. Its relationship to an engineering event should be
identifiable.

------------------------------------------------------------------------

## Version History and Milestone Relationship

Service `version-history/` folders record the evolution of a service.

Evidence may support a milestone recorded there, but the two have
different roles:

-   **version history** says what changed or what state was reached;
-   **evidence** demonstrates or substantiates a relevant part of that
    claim.

Avoid copying the same narrative into both locations.

------------------------------------------------------------------------

## Behavioural Experiment Evidence

As Aestimare develops, behavioural evidence will increasingly involve
repeated executions rather than isolated test runs.

The architectural principle is:

> A single execution is an observation. Repeated controlled executions
> become an experiment. Multiple experiments begin to characterise
> behaviour. Longitudinal evidence allows us to detect behavioural
> change.

For behavioural experiments, provenance should preserve enough
information to distinguish:

-   the task being assessed;
-   the controlled conditions;
-   individual executions;
-   Replay relationships;
-   repeated-run grouping;
-   observed divergence;
-   assessment method;
-   resulting assessment.

Repository evidence should contain selected durable examples or
published results, not necessarily the entire operational corpus.

------------------------------------------------------------------------

## Retention Guidance

Retain evidence when at least one of the following is true:

1.  It substantiates an important engineering milestone.
2.  It supports a published or externally visible claim.
3.  It is needed to reproduce or understand a significant behavioural
    finding.
4.  It records an unusual failure/divergence with continuing
    architectural value.
5.  It provides historical provenance that would otherwise be lost.

Evidence may be removed when it is:

-   routine and duplicated elsewhere;
-   an intermediate artefact with no independent value;
-   superseded by a more complete representation of the same evidence;
-   impossible to relate to a meaningful claim or execution context.

Deletion of operational data is a separate policy question from
repository housekeeping.

------------------------------------------------------------------------

## Naming Guidance

Prefer filenames that identify the subject and date/milestone without
requiring the file to be opened.

For example:

``` text
FIDUCIA_V0.7.0_TEST_RESULTS_2026-08-17.md
SERVIRE_END_TO_END_STARTUP_2026-08-17.png
AESTIMARE_EXPERIMENT_<name>_2026-08-xx.md
```

Do not encode a document's lifecycle status only in its filename. Use
the repository's document-status convention.

------------------------------------------------------------------------

## Evidence Is Not Assessment

A screenshot, trace, replay result or test output is evidence.

Statements such as "the model is stable", "the response is sufficient"
or "behaviour has degraded" are **assessments derived from evidence**.

Keeping that distinction explicit is essential to Lumen's architecture
and to the credibility of Reasoning Assurance.
