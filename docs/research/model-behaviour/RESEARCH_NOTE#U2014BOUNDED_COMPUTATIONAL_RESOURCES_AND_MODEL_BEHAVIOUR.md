# Research Note — Bounded Computational Resources and Model Behaviour

**Status:** Future Research  
**Target:** Post-M0.1 / Aestimare Research

## Research Question

Does executing the same model under different computational resource constraints result in observable changes in model behaviour?

The question is explicitly **not** whether constrained resources make inference slower.

The question is whether resource availability can materially affect the behaviour exhibited by the model.

## Experimental Principle

A controlled experiment could execute the same model under different resource envelopes while attempting to hold other variables constant.

For example:

```text
Model: Qwen

Execution A:
    tightly constrained resources

Execution B:
    normal resources

Execution C:
    generously available resources
```

Where possible, the following should remain constant:

- model and weights;
- quantisation;
- system prompt;
- user prompt;
- conversation context;
- inference parameters;
- tool availability;
- provider;
- software versions.

Trace would capture execution evidence and Aestimare could subsequently compare behavioural characteristics across executions.

## Behavioural Dimensions

Potential observations include changes in:

- completion behaviour;
- instruction adherence;
- response depth;
- context utilisation;
- consistency;
- tool selection;
- simplification;
- error behaviour;
- recovery behaviour;
- truncation;
- divergence across repeated executions.

Latency and throughput should also be recorded, but they are not themselves the primary subject of the experiment.

## Experimental Caution

Resource constraints do not necessarily alter model behaviour.

Some constraints may affect only execution speed while leaving inference effectively unchanged.

Other constraints may cause secondary changes such as:

- context reduction;
- memory-management changes;
- CPU/GPU offloading;
- altered batching;
- quantisation changes;
- provider configuration changes;
- inference failures or truncation.

The experimental design must therefore distinguish between:

> **resource availability itself**

and

> **configuration changes caused by insufficient resources.**

Both may be interesting, but they are different phenomena.

## Relationship to Bounded Rationality

Lumen's existing consideration of bounded reasoning and satisficing raises a related question.

Bounded rationality normally concerns reasoning or decision-making under limited information, time or cognitive resources.

For model systems, computational resources introduce another potential form of constraint.

This raises the broader research question:

> **Can externally imposed computational bounds produce measurable changes in model reasoning behaviour?**

Lumen's combination of controlled execution, Trace evidence, Replay and Aestimare potentially provides an environment in which this question can be investigated experimentally.

## Status

No implementation work is proposed for M0.1.

This note preserves the research question for future experimental design and Aestimare development.