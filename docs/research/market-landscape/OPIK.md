# Market Landscape Profile — Opik

**Product:** Opik by Comet  
**Reviewed:** 2026-08-26  
**Confidence:** Moderate — based on current public product/documentation review  
**Working classification:** Partial competitor / adjacent system / potential integration candidate

## Summary

Opik is an AI observability and evaluation platform focused on tracing, testing, evaluating, comparing, and improving LLM and agent applications.

It is one of the closer systems currently identified to Lumen at the **mechanism** level. It supports traces, datasets, experiments, behavioural/regression tests, output metrics, and evaluation of agent trajectories including intermediate steps and tool selection.

The current evidence nevertheless suggests a different centre of gravity: Opik primarily uses these mechanisms to determine and improve application quality and reliability, whereas Lumen's developing Reasoning Assurance direction uses repeated controlled evidence to characterise behaviour and behavioural change over time.

That distinction must be continually re-tested as both products evolve.

## Product Objective

Current Opik documentation frames the product around making agent behaviour observable, evaluating end-to-end and intermediate behaviour, detecting failures/regressions, comparing configurations, and iterating toward improved production performance.

Its evaluation model includes both datasets/metrics and behavioural test suites.

## Lumen Capability Overlap

| Capability | Lumen area | Preliminary overlap |
|---|---|---:|
| Execution tracing | Vestigare | High |
| Tool/intermediate-step tracing | Vestigare / Pontis | High |
| Dataset/test-case management | Fiducia / Aestimare research workflow | High |
| Repeatable experiments | Fiducia + Repetere + Aestimare | High |
| Output evaluation | Aestimare / Periti | High |
| Agent trajectory evaluation | Vestigare + Aestimare | High |
| Regression/behavioural testing | Aestimare | High |
| Prompt/model comparison | Moderari + Praebere + Aestimare | Medium–High |
| Historical replay | Repetere | Medium — requires deeper comparison |
| Prompt optimisation | No direct Lumen responsibility | Low conceptual overlap |
| Cross-session reasoning continuity | Lumen core | No equivalent established in this review |
| Longitudinal behavioural characterisation | Aestimare direction | Partial evidence only; requires continued review |
| Reasoning provenance across evolving work | Lumen core | No equivalent established in this review |

## Important Similarities

Opik demonstrates that several mechanisms important to Aestimare are already established market patterns:

- traces as experimental evidence;
- datasets of repeatable cases;
- experiments linking executions to dataset items;
- automated and custom evaluation metrics;
- comparison across models/configurations;
- assessment of intermediate agent steps rather than only final answers;
- behavioural assertions/regression testing.

Lumen should therefore avoid treating those mechanisms individually as differentiators.

## Current Boundary

A useful provisional distinction is:

**Opik:** observe → evaluate/test → compare → improve an AI application.

**Lumen:** preserve → reproduce → experiment → characterise → accumulate assurance evidence.

This is intentionally provisional. Opik already uses the language of agent behaviour and behavioural testing, so Lumen's differentiation cannot simply be "we assess behaviour".

The stronger candidate distinction is the role of **longitudinal accumulated evidence, continuity, provenance, controlled historical reproduction, and behavioural characterisation as an assurance record rather than principally an application-quality score.**

## What Opik Has That Lumen Should Learn From

- Mature evaluation/test-suite concepts.
- Established dataset and experiment UX.
- Agent trajectory evaluation.
- Production-oriented observability.
- Optimisation workflows.
- Broad integration ecosystem.

These should be studied before Lumen builds equivalent generic infrastructure.

## Potential Integration

Opik could eventually be considered an external evidence source. If an organisation already records agent traces and evaluation results in Opik, Aestimare should ideally be capable of considering that evidence through a Lumen-defined interface rather than requiring duplicate instrumentation.

This aligns with the Aestimare principle that external capabilities can contribute evidence without becoming the assessment itself.

## Positioning Implication

Do **not** position Lumen merely as an "AI observability and evaluation" product. Opik already occupies that space with substantial capability.

The comparison strengthens the need to define Reasoning Assurance in terms of the evidence relationship across observations, controlled reproduction, continuity, provenance, behavioural characterisation, and longitudinal change.

## Sources Reviewed

- Opik documentation: Evaluation Overview — https://www.comet.com/docs/opik/evaluation/overview
- Opik documentation: Evaluation Concepts — https://www.comet.com/docs/opik/evaluation/concepts
- Opik documentation: Evaluating Agents — https://www.comet.com/docs/opik/evaluation/evaluate_agents
- Opik documentation: Experiments — https://www.comet.com/docs/opik/reference/typescript-sdk/evaluation/experiments

## Open Questions

- How closely does Opik's current replay capability resemble Repetere's historical conversational replay and divergence/fork model?
- Does Opik preserve sufficient multi-session state to support continuity-oriented analysis?
- How far does its behavioural testing extend into repeated probabilistic behavioural characterisation rather than regression checking?
- What evidence/export APIs would make it practical as an Aestimare evidence provider?
