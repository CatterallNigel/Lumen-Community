# Engineering Principles

This document defines the engineering principles used throughout the Lumen project.

These principles apply equally to software development, architecture, documentation and research.

The objective is to ensure that engineering decisions remain evidence-based, transparent, reproducible and open to continual refinement as new knowledge is gained.

These principles should guide:

- Software development
- Architectural decisions
- Research activities
- Engineering documentation
- The Engineering Diary
- Future White Papers

---

## Transparency

Engineering decisions should be documented together with the evidence that supports them.

Where practical, important architectural decisions should reference the experiments, observations or discussions that led to those decisions.

---

## Reproducibility

Experiments should be described sufficiently that another engineer could reproduce the work and reasonably expect comparable results.

Where reproducibility is not possible, the reasons should be documented.

---

## Continuous Learning

Engineering knowledge is expected to evolve.

Previously accepted conclusions may be refined or replaced as new evidence becomes available.

Documentation should always reflect the current architectural understanding while Git preserves the historical evolution.

Changes to architectural terminology should be treated as engineering changes, not editorial changes.


Engineering knowledge is expected to evolve.

Previously accepted conclusions may be refined or replaced as new evidence becomes available.

Documentation should reflect current understanding rather than preserve historical assumptions.

---

## Evidence over Opinion

Engineering decisions should be supported by measurable observations wherever practical.

Personal preference should not outweigh experimental evidence.

Where evidence is insufficient, assumptions should be clearly identified as hypotheses.

---

## Architectural Vocabulary

Architectural terminology is part of the engineering process.

Every architectural concept should have one preferred definition.

One concept should have one preferred term, and one term should describe only one concept.

Changes to architectural vocabulary should be reflected consistently across the documentation set before new concepts are introduced.

---

## Hypothesis Status

Every research hypothesis should conclude with one of the following statuses.

| Status | Meaning |
|---------|---------|
| **Confirmed** | The experimental evidence supports the hypothesis. |
| **Partially Confirmed** | Some evidence supports the hypothesis, but additional investigation is required. |
| **Inconclusive** | The experiment did not produce sufficient evidence to either support or reject the hypothesis. |
| **Disproven** | The experimental evidence contradicts the hypothesis. |

### Notes

- A **Confirmed** hypothesis may still be refined by future research.
- **Partially Confirmed** indicates that evidence exists, but confidence is not yet sufficient to draw a final conclusion.
- **Inconclusive** results identify areas requiring improved experimentation or additional data.
- **Disproven** hypotheses are considered successful research outcomes because they eliminate incorrect assumptions and improve understanding.

