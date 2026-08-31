# Aestimare Examples

This folder contains worked ask/response trace examples for **Aestimare**, Lumen's behavioural assessment service.

The examples are intended to show how apparently simple model interactions can be converted into controlled, repeatable behavioural investigations. They preserve the original ask and response trace, identify observable behaviour, and propose variations that can distinguish a one-off error from a recurring behavioural characteristic.

## Purpose

Examples in this folder may be used to:

- develop and validate Aestimare assessment methods;
- identify candidate **Periti Assessores** and their assessment dimensions;
- design controlled repeated-execution experiments;
- compare behaviour across models, providers, prompts, and time;
- examine how a model responds to ambiguity, contradiction, correction, or user pressure;
- provide concrete reference cases for Aestimare documentation and development.

These are behavioural examples, not conventional model benchmarks. The objective is not merely to record whether an answer is correct. It is to examine how the model interpreted the ask, what assumptions it made, whether those assumptions were disclosed, how consistently it reasoned, and how its behaviour changed when challenged.

## Example Structure

Where applicable, each example should contain:

1. **Purpose** — why the interaction is useful for behavioural assessment.
2. **Observed trace** — the original asks and responses without retrospective correction.
3. **Interpretive context** — ambiguity, assumptions, or alternative valid readings.
4. **Observable behaviour** — what can be established directly from the trace.
5. **Assessment dimensions** — candidate behavioural characteristics to examine.
6. **Controlled prompt variants** — changes that isolate one variable at a time.
7. **Follow-up variants** — challenges or corrections used to examine response revision.
8. **Expected outcome classes** — possible response patterns and their interpretations.
9. **Experimental value** — what repeated executions may allow Aestimare to characterise.

Not every example must use every section. The structure should remain appropriate to the behaviour being investigated.

## Current Examples

| Example | Primary behavioural focus |
| --- | --- |
| [Days Containing the Letter D](AESTIMARE_ASK_RESPONSE_TRACE_EXAMPLE_DAY_LETTER_D.md) | Ambiguity detection, assumption disclosure, enumeration accuracy, verification, correction quality, and susceptibility to user agreement. |

## Naming Convention

Use descriptive uppercase Markdown filenames with the following general form:

```text
AESTIMARE_ASK_RESPONSE_TRACE_EXAMPLE_<SUBJECT>.md
```

For example:

```text
AESTIMARE_ASK_RESPONSE_TRACE_EXAMPLE_DAY_LETTER_D.md
```

The filename should describe the interaction itself rather than pre-judge the behaviour observed. Behavioural conclusions may change as more models and executions are assessed.

## Evidence Discipline

Each example should distinguish clearly between:

- **Observation** — what is present in the recorded trace.
- **Interpretation** — what that observation may indicate.
- **Hypothesis** — a behavioural proposition requiring repeated controlled evidence.
- **Characterisation** — a conclusion supported by multiple experiments or longitudinal evidence.

A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows behavioural change to be detected.

## Adding an Example

When adding a new example:

1. Preserve the original wording, capitalisation, punctuation, and response order.
2. Record relevant execution metadata when available, including model, provider, configuration, system prompt mode, and date.
3. Separate directly observable facts from inferred causes.
4. Identify alternative reasonable interpretations of the ask.
5. Propose prompt variants that change only one meaningful variable at a time.
6. Include follow-up asks where revision behaviour is part of the investigation.
7. Add the example to the **Current Examples** table in this README.

## Scope

This folder contains explanatory and experimental examples. Formal Peritus specifications, scoring rules, schemas, and implementation details should remain in their respective Aestimare or `periti_assessores` documentation locations.

