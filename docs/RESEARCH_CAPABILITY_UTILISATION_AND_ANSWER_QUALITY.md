# RESEARCH_CAPABILITY_UTILISATION_AND_ANSWER_QUALITY.md

# Capability Utilisation and Answer Quality
## Evaluating How Well a Model Uses Its Own Capability

**Author:** Nigel Catterall  
**Project:** Lumen Research  
**Date:** 31 July 2026

---

# Abstract

Most benchmarking research attempts to answer one question:

> Which model is better?

This research proposes a different question.

> Given a fixed model, how well does that model utilise its own capability?

Rather than comparing different models, this research investigates the variability exhibited by a single model operating under an identical environment, identical prompt, identical tools and identical orchestration.

Initial observations from Lumen indicate that the same model can produce substantially different reasoning paths, execution strategies and final answers while operating within an apparently identical execution environment.

This suggests that a model's capability should not be viewed as the answer it produces, but rather as the upper bound of the answers it is capable of producing.

The quality of an individual answer therefore represents only one sample from that capability.

---

# Background

Traditional LLM evaluation focuses on:

- benchmark scores
- accuracy
- pass/fail measurements
- comparisons between models

These approaches assume that a single answer reasonably represents the capability of the model.

However repeated executions suggest otherwise.

During testing of Lumen's execution observability, repeated executions of the same task demonstrated noticeable variation despite no intentional changes to:

- model
- prompt
- context
- tools
- orchestration
- execution environment

The model therefore exhibited variability in both execution behaviour and answer quality.

---

# Experimental Setup

Environment remained effectively constant.

- Lumen
- qwen2.5-coder:14b-32k
- identical prompt
- identical source file
- identical tool availability
- identical orchestration
- identical hardware

Prompt:

> Read `src/ef_social_discovery/api/routes/operations.py` in full and then summarise it.

---

# Observed Behaviour

Multiple executions produced different execution paths.

Examples included:

```
read

↓

answer
```

```
read

↓

write summary.txt

↓

answer
```

```
read

↓

write summary.txt

↓

read summary.txt

↓

answer
```

Earlier experiments also demonstrated:

```
read

↓

repeated read

↓

extended reasoning

↓

answer
```

The execution strategy therefore varied despite identical inputs.

---

# Answer Variance

The resulting summaries also differed.

Observed differences included:

- answer length
- structure
- ordering
- completeness
- terminology
- emphasis

Some responses consisted of only a brief paragraph.

Others produced structured functional analyses.

Others expanded helper functions and architectural responsibilities.

Importantly, all were factually acceptable.

However they clearly represented different levels of answer quality.

---

# Stable Semantic Understanding

Despite the differing outputs, the model consistently identified the same major concepts.

These included:

- pipeline monitoring
- ingestion integrity
- queue health
- manual refresh operations
- thread digest processing
- engagement management
- Reddit context processing
- helper/default-limit functions

This suggests that semantic understanding remained relatively stable while expression of that understanding varied.

---

# Capability versus Answer

One of the most important observations is that capability and answer are not equivalent.

Instead they appear to have the following relationship.

```
Model Capability

        │

        ▼

Potential Answer Space

        │

        ▼

Chosen Reasoning Path

        │

        ▼

Produced Answer
```

The produced answer therefore represents only one possible realisation of the model's capability.

---

# Capability Ceiling

Repeated executions demonstrated that Qwen was capable of producing answers of substantially different quality.

This leads to an important conclusion.

The poorer answers cannot represent the true capability of the model because the model subsequently demonstrated significantly better answers.

The capability therefore remains constant while realised performance varies.

Conceptually:

```
Capability Ceiling

100%

│

├── Execution A
│       45%
│
├── Execution B
│       61%
│
├── Execution C
│       83%
│
└── Execution D
        90%
```

The capability does not change.

Only the proportion of that capability realised in each execution changes.

---

# Capability Utilisation

This suggests a new concept.

Instead of asking:

> How capable is the model?

we ask:

> How effectively did the model utilise its capability during this execution?

Capability therefore becomes relatively static.

Capability utilisation becomes dynamic.

```
Capability

↓

Capability Utilisation

↓

Answer Quality
```

Answer quality becomes the observable consequence of capability utilisation.

---

# Why Does Variability Occur?

Although identical inputs were supplied, the model still produced different outputs.

Possible contributors include:

- differing reasoning trajectories
- attention allocation
- planning decisions
- stopping decisions
- tool selection
- summarisation strategy
- token probability distributions

Even with a deterministic environment, multiple valid reasoning paths appear possible.

---

# Lumen as an Observability Platform

Prior to Lumen these behavioural differences were largely invisible.

Lumen now records:

- execution timeline
- reasoning route
- tool usage
- source coverage
- persistence
- checkpoints
- execution duration

This makes previously hidden behavioural variation observable.

The Operational Intelligence dashboard therefore evolves beyond infrastructure monitoring.

It becomes an instrumentation platform for studying model behaviour.

---

# Process Variance

Two successful executions may exhibit very different execution characteristics.

Examples include:

```
read

↓

answer
```

versus

```
read

↓

write summary

↓

read summary

↓

answer
```

Both may ultimately produce acceptable answers.

However the execution efficiency differs.

---

# Answer Variance

Similarly two successful executions may differ in:

- completeness
- organisation
- clarity
- level of detail
- usefulness

Traditional benchmarking records both as success.

Lumen reveals that the quality of successful executions is itself variable.

---

# A New Evaluation Question

Traditional evaluation asks:

> Which model performs best?

This research proposes an alternative.

> How consistently does a model reach its own potential?

This shifts evaluation away from inter-model comparison and toward intra-model consistency.

---

# Capability Utilisation Efficiency

One possible future metric is:

```
Capability Utilisation Efficiency (CUE)

=

Observed Answer Quality

──────────────────────────────

Estimated Capability Ceiling
```

Determining the capability ceiling remains an open research problem.

However Lumen provides a possible approach.

Repeated executions of equivalent tasks allow empirical estimation of the highest demonstrated capability of a model.

Subsequent executions may then be evaluated relative to that demonstrated capability.

---

# Implications

This research suggests that future orchestration systems should not simply accept the first answer produced.

Instead an orchestrator may ask:

- Was this likely the best answer the model could produce?
- Is further reasoning justified?
- Should the answer be expanded?
- Should the reasoning be verified?
- Should the model critique its own work?

In this view, orchestration becomes an optimisation process rather than simply a routing process.

---

# Research Hypothesis

The central hypothesis emerging from this work is:

> Model capability and answer quality are not equivalent.

Instead:

- capability defines the upper bound of possible performance;
- each execution realises only part of that capability;
- answer quality depends upon how effectively that capability is utilised during execution.

Consequently, two executions of the same model under identical conditions may legitimately produce different answers of differing quality without any change in the underlying capability of the model itself.

---

# Future Research

Areas for further investigation include:

- estimating model capability ceilings
- measuring execution consistency
- identifying execution routes that consistently produce higher quality answers
- measuring answer quality relative to demonstrated capability
- determining whether orchestration can improve capability utilisation without changing the underlying model
- comparing capability utilisation across different model families
- correlating reasoning path complexity with answer quality
- investigating whether repeated self-reflection increases capability utilisation
- developing statistical measures of answer variance over repeated executions

---

# Conclusion

Today's observations suggest that the question of "how good is the model?" may be less useful than previously believed.

A more meaningful question may be:

> How effectively did the model utilise the capability it already possesses?

Lumen's observability makes this question measurable.

Rather than evaluating only the final answer, it becomes possible to evaluate the quality, consistency and efficiency of the reasoning process itself.

This represents a shift from evaluating model capability to evaluating capability utilisation.

If this hypothesis proves correct, orchestration systems such as Lumen may ultimately improve practical model performance not by changing the model, but by helping the model more consistently reach its own demonstrated potential.