# RESEARCH_MODEL_CAPABILITY.md

# Research: Measuring Model Capability vs Answer Capability

## Status

Research Proposal

---

# Abstract

Current benchmark methodologies primarily measure the correctness of a model's response to a prompt. They do not attempt to determine whether the response represents the full extent of the model's knowledge or merely the amount of knowledge the model chose to express.

This document explores the distinction between **knowledge**, **expressed answers**, and **maximum achievable responses**, and discusses how these may be estimated probabilistically through repeated interaction.

---

# Three Distinct Concepts

## 1. Knowledge Capability

**Definition**

The total knowledge or reasoning ability that the model (or person) possesses at a particular point in time.

This represents everything the model is capable of using if provided with sufficient context, prompting, time and reasoning budget.

Knowledge Capability is largely unobservable.

---

## 2. Answer Capability

**Definition**

The amount of available knowledge that the model chooses (or is able) to express in response to a particular prompt.

A model rarely produces everything it knows.

Its response may be limited by:

* prompt wording
* available context
* token budget
* inference budget
* assumptions about the user's intent
* preference for brevity
* safety or policy constraints
* uncertainty

Therefore:

```
Answer Capability ≤ Knowledge Capability
```

---

## 3. Maximum Answer Capability

**Definition**

The best answer the model could reasonably produce **given only the information available to it**.

This is subtly different from Knowledge Capability.

Knowledge Capability asks:

> *What does the model know?*

Maximum Answer Capability asks:

> *Given everything the model has been told (or has ingested), what is the best answer it is capable of producing?*

A poor answer may therefore arise because:

* the model lacks knowledge,
* the prompt lacks sufficient information,
* the model misunderstood the request,
* or the model simply chose not to express its full reasoning.

These situations should not be treated as equivalent.

---

# Human Analogy

Consider asking a highly experienced hardware technician a question.

They provide a short answer.

Later another expert gives a much richer explanation.

Returning to the original technician reveals two possibilities.

## Case A

They reply:

> "That's interesting—I didn't know that."

Their original answer represented approximately 100% of their available knowledge.

---

## Case B

They reply:

> "Yes, that's true. In fact..."

They then spend several minutes expanding on the subject.

Their original answer did **not** represent their full capability.

Instead it reflected a decision regarding:

* brevity
* assumed audience
* effort
* conversational efficiency

Humans behave this way constantly.

Large language models appear to exhibit similar behaviour.

---

# The Central Research Question

Instead of asking:

> Was the answer correct?

Ask:

> Was this the best answer the model was capable of producing?

These are fundamentally different questions.

---

# Measuring Expressed Capability

A useful conceptual metric may be:

```
Expressed Capability Ratio (ECR)

=
Observed Answer Quality
------------------------
Estimated Maximum Answer Capability
```

Where:

An ECR approaching **1.0** suggests the model expressed most of its available capability for the information it possessed.

A lower ECR suggests that additional capability may remain unexpressed.

The denominator cannot be directly observed and must therefore be estimated.

---

# Estimating Capability

Direct measurement is impossible.

Instead, capability must be inferred probabilistically.

Possible evidence includes:

* follow-up questions
* requests for greater depth
* alternative prompt wording
* counter-examples
* contradictory evidence
* edge cases
* reasoning consistency
* correction behaviour
* cross-session consistency
* auditor model evaluation

Each observation updates confidence rather than producing a deterministic score.

---

# Behaviour Over Time

Capability should not be inferred from a single response.

Instead:

```
Interaction
    ↓

Observation
    ↓

Evidence
    ↓

Updated Probability
```

Trust becomes an evolving estimate built from many observations.

This mirrors how humans evaluate other experts.

---

# Probabilistic Trust

Rather than stating:

```
Model Capability = 92%
```

a more realistic representation is:

```
P(Model can produce a substantially better answer
  | observations)
```

or

```
P(This response represents the model's maximum answer capability
  | observations)
```

Trust therefore becomes a Bayesian-style estimate that evolves as new evidence is collected.

---

# Implications for Lumen

Lumen is uniquely positioned to explore this area because it maintains continuity across interactions.

Rather than evaluating isolated answers, Lumen can accumulate behavioural evidence across sessions.

Potential future metrics include:

* Knowledge Capability (estimated)
* Answer Capability
* Maximum Answer Capability (estimated)
* Expressed Capability Ratio (ECR)
* Trust Score
* Consistency Score
* Expansion Score (how much additional knowledge is revealed after follow-up questioning)
* Self-Correction Score
* Cross-Session Stability

These measurements would not attempt to determine absolute intelligence.

Instead they would estimate, probabilistically, whether the model is consistently producing the best answers it is capable of producing with the information available.

---

# Research Questions

* Can repeated questioning estimate latent capability?
* Can follow-up prompts distinguish between missing knowledge and abbreviated answers?
* How stable are capability estimates across sessions?
* Does prompt engineering significantly alter the Expressed Capability Ratio?
* Can independent auditor models improve confidence in capability estimates?
* How quickly does trust converge after repeated interactions?
* Can Lumen maintain longitudinal capability profiles for different models?

---

# Initial Conclusion

The quality of a response should not be treated as a direct measurement of model capability.

Instead, capability should be viewed as a latent variable that can only be estimated through repeated observation.

Lumen's persistent memory, continuity, and behavioural history provide an opportunity to build probabilistic trust models based on evidence accumulated over time, rather than on isolated responses.

This shifts evaluation from asking *"Was this answer correct?"* to asking *"Given everything the model knew and everything it had been told, how likely is it that this was the best answer it could have produced?"*
