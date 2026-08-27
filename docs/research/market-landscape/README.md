# Lumen Market Landscape Research

**Status:** Living research area  
**Created:** 2026-08-26

## Purpose

This directory tracks products, projects, and platforms whose capabilities overlap with, resemble, complement, or potentially compete with Lumen.

The purpose is not conventional competitor marketing. It is to use external systems as evidence for sharpening Lumen's definition:

- which capabilities are common in the market;
- which Lumen mechanisms overlap with established products;
- where apparently similar mechanisms serve different objectives;
- which capabilities Lumen should integrate rather than reproduce;
- where genuine differentiation exists;
- and which claims about Lumen can be supported by evidence rather than assumption.

The central question is:

> **What does Lumen need to be uniquely good at, and how can we demonstrate that distinction?**

## Classification

An entry may be classified as one or more of:

- **Direct competitor** — substantially addresses the same problem for the same users.
- **Partial competitor** — overlaps materially with part of Lumen but not the complete objective.
- **Adjacent system** — operates in a neighbouring problem domain with useful functional overlap.
- **Complementary system** — could provide evidence or capability to Lumen without being replaced by it.
- **Integration candidate** — exposes data or capabilities that could potentially be consumed through a Lumen-defined boundary.

These classifications are hypotheses and may change as products and Lumen evolve.

## Overlap Scale

| Rating | Meaning |
|---|---|
| None | No meaningful capability overlap identified. |
| Low | Similar terminology or isolated mechanism, but materially different responsibility. |
| Medium | Several related capabilities or one substantial shared mechanism. |
| High | Significant functional overlap across an important Lumen responsibility. |
| Very High | Similar mechanisms and substantially similar product objective. |

A high functional-overlap rating does **not** imply that two systems have the same purpose.

## Assessment Dimensions

Each profile should consider:

| Dimension | Question |
|---|---|
| Product objective | What problem does the system say it solves? |
| Category | Observability, evaluation, orchestration, governance, optimisation, continuity, etc. |
| Lumen relationship | Competitor, adjacent, complementary, integration candidate? |
| Functional overlap | Which Lumen services or mechanisms overlap? |
| Overlap strength | None / Low / Medium / High / Very High |
| Similarities | What do both systems actually do? |
| Architectural difference | Where do responsibilities or boundaries differ? |
| Missing from Lumen | What mature capability exists externally that Lumen does not needlessly need to reproduce? |
| Missing from them | Which Lumen responsibilities are not evidenced in the external system? |
| Integration potential | Could Lumen consume traces, scores, evaluations, or other evidence? |
| Positioning implication | What does the comparison teach us about describing Lumen? |
| Confidence | Preliminary / Moderate / High, with evidence date. |

## Important Distinction: Mechanism vs Objective

A recurring risk is treating a shared mechanism as proof that two products solve the same problem.

For example, tracing, datasets, repeated experiments, evaluation, prompt comparison, and replay are increasingly common AI-engineering capabilities. Lumen therefore should not claim uniqueness merely because it implements one of these mechanisms.

The comparison must ask why the mechanism exists and what conclusion the system is designed to support.

A useful working distinction is:

- **Observability:** what happened?
- **Evaluation:** did an execution satisfy defined criteria?
- **Optimisation:** how can performance against those criteria be improved?
- **Governance:** should an action be allowed, modified, or blocked?
- **Reasoning Assurance:** what accumulated evidence supports confidence in how behaviour develops, reproduces, varies, and changes?

This distinction is a research hypothesis, not a marketing claim. The market landscape should continually test it.

## Current Profiles

- [Opik](OPIK.md)
- [LangSmith](LANGSMITH.md)
- [Langfuse](LANGFUSE.md)
- [Arize Phoenix](ARIZE_PHOENIX.md)
- [Market Capability Matrix](MARKET_CAPABILITY_MATRIX.md)

## Other Systems to Investigate

Capture candidates here before spending significant research time on them.

| System | Initial reason for interest | Status |
|---|---|---|
| Weights & Biases Weave | Tracing, evaluation and experimentation | To investigate |
| Braintrust | Evaluation, experiments and production observability | To investigate |
| Galileo | AI observability and evaluation | To investigate |
| Helicone | LLM observability and gateway capabilities | To investigate |
| OpenLLMetry / Traceloop | OpenTelemetry-based LLM observability | To investigate |
| MLflow GenAI | Tracing, evaluation and model/application lifecycle | To investigate |

Addition to this list does not imply competitive overlap.

## Maintenance Principle

This should remain a lightweight, evidence-led research stream. We should add systems as they become relevant rather than attempting an exhaustive market survey.

When a comparison materially changes our understanding of Lumen, the conclusion should be reflected in the appropriate architecture, research, service, or positioning documentation rather than remaining only in this directory.
