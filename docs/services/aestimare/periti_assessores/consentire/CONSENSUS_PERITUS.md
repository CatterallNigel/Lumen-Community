# Consensus Peritus

**Status:** Concept / Future Peritus\
**Service:** Aestimare\
**Purpose:** Independent multi-model assessment, convergence analysis, and satisficing-based confidence determination

---

## Overview

The **Consensus Peritus** provides Aestimare with a mechanism for obtaining a second opinion, or multiple independent second opinions, when an assessment requires interpretation or reasoning that cannot reasonably be implemented deterministically.

This is no different in principle from the human practice of seeing a specialist and asking for a second opinion.

Where an Aestimare assessment requires interpretation or reasoning that cannot reasonably be implemented deterministically, the Consensus Peritus submits the same evidence independently to multiple suitably equivalent LLMs.

The resulting assessments are then compared for convergence, disagreement, uncertainty, and evidential support.

The objective is **not necessarily consensus**.

The objective is to determine whether the available independent assessments provide **sufficient confidence to satisfy the requirements of the assessment**.

This follows a satisficing rather than optimisation model:

> An assessment is sufficient when additional independent assessment is unlikely to materially alter the conclusion.

---

## Motivation

Aestimare may require LLM capability for some forms of behavioural assessment.

This creates an immediate problem:

> If an LLM is used to assess the behaviour or output of another LLM, who assesses the assessor?

In the human world, this is addressed by consulting a specialist and, where necessary, seeking a second opinion. The second opinion does not automatically establish truth, but it provides an independent perspective that may confirm, qualify, or challenge the initial assessment.

Simply introducing another model does not remove the problem. It merely transfers authority from one probabilistic system to another.

The Consensus Peritus addresses this by replacing reliance upon a single assessor with **independent model triangulation**.

Rather than:

```text
Evidence
   |
   v
LLM Assessor
   |
   v
Assessment
```

the assessment becomes:

```text
                    +--> Assessor A --> Assessment A
                    |
Evidence -----------+--> Assessor B --> Assessment B
                    |
                    +--> Assessor C --> Assessment C
                                   |
                                   v
                        Convergence Analysis
                                   |
                                   v
                       Sufficient Confidence?
```

Each assessor produces an independent judgement from the same underlying evidence.

Aestimare then evaluates the relationship between those judgements.

---

## Model Equivalence

For the purposes of the Consensus Peritus, **equivalent models do not need to be identical models**.

Model equivalence is task-specific.

> Two or more models are equivalent for an assessment task when they possess sufficiently comparable capability, independence, and task suitability that each can independently evaluate the same evidence to the required standard.

Equivalence therefore does not imply:

- identical architecture;
- identical provider;
- identical benchmark performance;
- identical model size;
- identical training data;
- identical responses.

Indeed, excessive similarity between assessors may reduce the evidential value of agreement.

The Consensus Peritus should prefer **functional equivalence with useful diversity** over model identity.

---

## Equivalence Dimensions

Selection of assessors should consider at least two separate characteristics.

### Task Equivalence

Can each model competently perform the specific assessment being requested?

A model may be equivalent for one assessment task but not another.

For example, models may be equivalent for:

- semantic comparison;
- contradiction detection;
- evidence summarisation;

while not being equivalent for:

- specialist legal interpretation;
- mathematical verification;
- domain-specific scientific reasoning.

Equivalence must therefore be established against the **assessment task**, not against a general concept of model capability.

### Assessment Independence

Are the selected models sufficiently independent that agreement provides meaningful additional evidence?

Three closely related models may exhibit correlated failure because they share:

- architecture;
- training material;
- alignment techniques;
- provider-specific behaviour;
- systematic biases;
- common benchmark optimisation.

Where practical, Aestimare should therefore favour assessors from different model families or providers when this increases meaningful independence.

---

## Independent Assessment First

Assessors should initially operate independently.

Assessor B and Assessor C should **not** normally be shown Assessor A's conclusion before producing their own assessment.

Doing so could introduce:

- anchoring;
- confirmation bias;
- framing effects;
- convergence caused by imitation rather than evidence.

The initial stage should therefore be:

```text
Evidence --> A
Evidence --> B
Evidence --> C
```

with no assessor having access to another assessor's judgement.

Only after all initial assessments have been recorded should their conclusions be compared.

---

## Convergence Analysis

The Consensus Peritus should not reduce assessment to simple majority voting.

For example:

| Assessor | Conclusion                            | Confidence |
| -------- | ------------------------------------- | ---------- |
| A        | Behaviour changed materially          | High       |
| B        | Behaviour changed materially          | High       |
| C        | Behaviour changed, but not materially | Medium     |

A simple voting mechanism produces:

```text
2 versus 1
Therefore: material change
```

This discards potentially important evidence.

Instead, the Consensus Peritus should examine:

- areas of agreement;
- areas of disagreement;
- evidence cited by each assessor;
- confidence expressed by each assessor;
- assumptions made;
- interpretation differences;
- materiality of disagreement;
- whether disagreement affects the final conclusion.

Assessor C's disagreement may reveal an ambiguity or evidential weakness that the other assessors overlooked.

Therefore:

> **Disagreement is evidence, not failure.**

---

## Satisficing

The Consensus Peritus should not require universal agreement.

Its stopping condition should instead be based upon **sufficient confidence**.

A possible governing principle is:

> Assessment satisficing occurs when independent, suitably equivalent assessors demonstrate sufficient convergence that further assessment is unlikely to materially alter the conclusion.

This creates an important distinction:

```text
Consensus != Truth
```

and:

```text
Disagreement != Failure
```

The objective is instead:

```text
Accumulated Evidence
        +
Independent Assessment
        +
Measured Convergence
        +
Known Uncertainty
        |
        v
Sufficient Confidence
```

---

## Indeterminate Results

The Consensus Peritus must be capable of returning **indeterminate**.

If suitably equivalent assessors repeatedly disagree in a material way, the correct result may be:

> The available evidence does not support a stable assessment at the required confidence level.

This is preferable to manufacturing certainty through majority voting.

An indeterminate result may itself be a significant Aestimare finding.

Repeated disagreement could indicate:

- ambiguous evidence;
- insufficient evidence;
- unstable model interpretation;
- poorly defined assessment criteria;
- assessor capability mismatch;
- a genuinely uncertain behavioural characteristic.

---

## Additional Assessment

The Consensus Peritus may optionally determine whether another independent assessment would provide useful information.

For example:

```text
A agrees strongly
B agrees strongly
C differs slightly
```

may already satisfy the assessment threshold.

Whereas:

```text
A strongly supports X
B strongly supports Y
C is uncertain
```

may justify obtaining another independent assessment.

The question therefore becomes:

> Would another assessor be reasonably likely to materially change our confidence in the conclusion?

If not, further assessment has diminishing value and the satisficing threshold has been reached.

This also introduces an economic dimension: assessment confidence has a computational and financial cost.

---

## Provenance Requirements

Every Consensus Peritus execution should preserve:

- model/provider identity;
- model version where available;
- assessment prompt or assessment specification;
- evidence supplied to the assessor;
- assessor response;
- confidence or uncertainty;
- cited evidence;
- execution parameters;
- execution timestamp;
- relationship to the originating Aestimare assessment;
- convergence analysis;
- final satisficing decision.

The individual assessments must remain available even after Aestimare produces an aggregate conclusion.

The aggregate conclusion must never replace its underlying evidence.

---

## Relationship to Aestimare

The Consensus Peritus does not determine whether an AI system is objectively "correct".

It provides evidence concerning the stability and supportability of an Aestimare assessment.

The broader progression remains:

```text
Single execution
    |
    v
Observation

Repeated controlled executions
    |
    v
Experiment

Multiple experiments
    |
    v
Behavioural evidence

Independent assessment
    |
    v
Triangulation

Convergence analysis
    |
    v
Satisficing threshold

Longitudinal evidence
    |
    v
Behavioural characterisation
```

This preserves a core Aestimare principle:

> **A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows us to detect behavioural change.**

The Consensus Peritus extends this principle into the assessment layer.

A single AI judgement is an assessment observation.

Multiple independent judgements provide evidence about the reliability of that assessment.

---

## Design Principle

The Consensus Peritus should never claim:

> Three models agree, therefore the conclusion is true.

Instead it should be capable of stating:

> Multiple suitably equivalent and independently selected assessors evaluated the same evidence, providing a second-opinion process analogous to consulting multiple specialists. Their assessments converged sufficiently that further assessment is unlikely to materially alter the conclusion at the required confidence threshold.

This changes the role of AI within Aestimare.

Aestimare is not asking one AI to certify another AI.

It is:

> **Using multiple independently selected assessors, preserving their individual judgements and provenance, measuring their convergence and disagreement, and stopping when the accumulated evidence is sufficient for the stated purpose.**

---

## Open Questions

The implementation will need to determine:

- How is task equivalence established and recorded?
- How should assessor independence be measured?
- What constitutes sufficient model diversity?
- Should the minimum assessor set normally be three?
- How are confidence values normalised between models?
- How should disagreement be classified and weighted?
- How is the satisficing threshold defined for different assessment types?
- When should an additional assessor automatically be invoked?
- Should assessor selection itself be deterministic?
- How should correlated model failures be detected?
- How should assessment cost influence the satisficing threshold?
- Should the Consensus Peritus assess its own historical reliability?
- Can longitudinal evidence establish which assessor combinations provide the strongest predictive stability?

These questions should be answered experimentally rather than assumed during initial implementation.
