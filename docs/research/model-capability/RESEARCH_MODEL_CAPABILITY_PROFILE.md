# Model Capability Profile

**Status:** Research

**Version:** Draft 0.1

---

# Purpose

Lumen's objective is not to determine which AI model is "best".

Instead, Lumen seeks to determine **how effectively a model has performed relative to its own capabilities**.

Traditional benchmarking compares different models against one another.

Lumen proposes an alternative approach.

> *How close did this model come to producing the best answer it is capable of producing?*

---

# Motivation

Current model comparisons frequently answer questions such as:

- Which model is more accurate?
- Which model reasons better?
- Which model writes better code?

These comparisons are valuable when selecting a model.

However, they do not answer an equally important engineering question.

> *Did this particular model perform well?*

For example:

A Qwen 14B model may produce a weaker answer than a GPT-5 model.

That does **not** necessarily imply Qwen performed poorly.

It may have produced an answer that is very close to the maximum quality that model is capable of achieving.

Conversely, a much larger model may produce a disappointing answer despite having significantly greater capability.

Lumen therefore proposes evaluating models relative to themselves rather than relative to other models.

---

# Principle

Each model has an observable capability envelope.

Lumen seeks to estimate where an individual response lies within that envelope.

```
                Maximum Capability
                       ▲
                       │
Excellent Response     │
                       │
Good Response          │
                       │
Average Response       │
                       │
Poor Response          │
                       │
Minimum Capability     ▼
```

The objective is to estimate the position of a response within that envelope.

---

# Why This Matters

This changes how models can be selected.

Example:

| Model | Capability | Response |
|--------|-----------:|---------:|
| Qwen 14B | 90% of capability | Good |
| GPT-5 | 45% of capability | Moderate |

Although GPT-5 may have a much higher theoretical capability, the current response from Qwen may represent a significantly better utilisation of the model.

This creates opportunities for intelligent orchestration.

---

# Adaptive Model Escalation

If Lumen determines that:

- the current model has performed close to its maximum capability, and
- the user's objective has still not been achieved,

then moving to a larger or more capable model becomes a rational engineering decision.

Rather than simply requesting:

> "Use GPT-5."

Lumen can reason:

> "Qwen has produced an answer close to its expected capability ceiling. Achieving substantially better results is unlikely without using a more capable model."

This provides an objective basis for model escalation.

---

# The Fundamental Research Question

The central challenge is:

> **How can Lumen estimate how close a response is to the maximum capability of the model that produced it?**

This is currently an open research problem.

---

# Possible Sources of Evidence

No single measurement is likely to be sufficient.

Instead, capability estimation may emerge from combining multiple independent observations.

Possible evidence includes:

## Continuity

- checkpoint stability
- checkpoint convergence
- cognitive evolution
- completeness of Final Cognitive Checkpoint

---

## Structural Quality

- architectural synthesis
- abstraction quality
- consistency
- coherence

---

## Coverage

- source coverage
- function coverage
- subsystem coverage
- concept coverage

---

## Evolution

- checkpoint evolution
- assumption refinement
- reduction of unknowns
- convergence of understanding

---

## Behaviour

- contradiction rate
- hallucination rate
- self-correction
- confidence stability

---

## External Validation

Future versions of Lumen may incorporate independent reviewer models.

These reviewers would not determine capability directly.

Instead they would provide additional evidence contributing to the capability estimate.

---

# Capability Profile

Over time, Lumen may build a statistical profile for each supported model.

Example:

```
Qwen 14B

Architecture synthesis      High
Function recall             Excellent
Cross-file reasoning        Moderate
Consistency                 High
Hallucination resistance    Good
Checkpoint stability        Excellent
```

This profile would evolve as more benchmark data becomes available.

---

# Research Direction

Rather than assigning a simple score, Lumen should seek to estimate a probability.

Example:

```
Estimated Capability Utilisation

91%

Confidence

Moderate
```

This recognises that capability estimation is inherently uncertain.

---

# Future Questions

The following questions remain open.

- Can capability be measured objectively?
- Is capability domain-specific?
- Does capability change with prompt design?
- Does capability change with context size?
- Can capability be estimated without external reviewers?
- How many benchmark runs are required?
- Can capability be transferred between model versions?
- Does capability remain stable over time?
- Can different reasoning tasks produce different capability envelopes?

---

# Relationship to Lumen

Model Capability Profiles extend Lumen beyond continuity management.

Continuity preserves the observable cognitive state of a model.

Capability Profiles seek to characterise how effectively that cognitive state has been utilised.

Together they enable evidence-based model orchestration rather than model selection based solely on benchmark rankings.

---

# Conclusion

The Model Capability Profile is an exploratory research direction.

Its objective is not to determine whether one model is better than another.

Instead it seeks to answer a more fundamental engineering question:

> **Did this model produce a result close to the best it is realistically capable of producing?**

If that question can be answered with sufficient confidence, Lumen gains the ability to make objective decisions about when to continue using the current model and when escalation to a more capable model is justified.