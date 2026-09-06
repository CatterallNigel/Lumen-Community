# Moribus Peritus — Behavioural Assessment Research

**Status:** Research / Future Peritus  
**Service:** Aestimare  
**Peritus:** Moribus Peritus  
**Domain:** Behavioural assessment  
**Initial model constraint:** Open-weight models ≤14B parameters

---

## Overview

The **Moribus Peritus** is a proposed Aestimare specialist assessor concerned with the observable behaviour of AI models.

Its methodological foundation draws upon established behavioural psychology and behavioural science without assuming that an AI model possesses human psychological states.

The purpose is not to create or claim a new discipline of "AI psychology".

Instead, the research question is:

> Can established methods for observing, experimenting upon, and characterising behaviour provide useful methods for assessing the observable behaviour of probabilistic AI systems?

Behavioural psychology provides an existing body of knowledge concerning:

- controlled observation;
- stimulus and response;
- repeated experimentation;
- behavioural consistency;
- contextual effects;
- framing;
- adaptation;
- ambiguity;
- behavioural variation;
- longitudinal behavioural change;
- uncertainty;
- confounding variables.

Many of these concepts are potentially applicable to the behavioural evidence already collected by Lumen and assessed by Aestimare.

---

## Why "Behavioural" Rather Than "Human Behavioural"

The proposed Peritus deliberately uses **Behavioural** rather than **Human Behavioural Psychology**.

Behavioural science is not exclusively concerned with humans. Established behavioural disciplines also study animal behaviour and other observable systems.

AI models are neither humans nor animals.

However, many AI systems are trained upon human-produced information and language and interact with humans through representations derived from human communication.

Human behavioural research may therefore provide useful experimental methods and descriptive concepts.

This does **not** imply equivalence between human cognition and model computation.

The distinction should remain explicit:

> Moribus characterises observable model behaviour. It does not infer human psychological states within a model.

---

## Behaviour Versus Internal State

Moribus should distinguish carefully between an **observed behavioural effect** and an explanation of the internal mechanism producing it.

For example, an experiment may establish that introducing a particular value or statement into context systematically shifts subsequent model responses relative to a control.

It may therefore be reasonable to describe:

> An anchoring-like behavioural effect was observed under the experimental conditions.

It would not necessarily be justified to conclude:

> The model experiences the human cognitive bias of anchoring.

The former describes observable evidence.

The latter makes a claim concerning an internal process that the evidence may not establish.

This distinction should be fundamental to Moribus.

---

## Initial Research Hypothesis

Aestimare may eventually require LLM capability for assessments that cannot reasonably be performed using deterministic algorithms alone.

This raises a useful research question:

> Would a model specifically trained or adapted using behavioural psychology or behavioural-science material produce better behavioural assessments than a general-purpose model given the same assessment methodology?

There are several possible levels of specialisation.

### Level 1 — Prompt Specialist

```text
General-purpose model
        +
Moribus system prompt
        +
Structured assessment protocol
```

The behavioural methodology is expressed primarily through the system and assessment prompts.

### Level 2 — Knowledge Specialist

```text
General-purpose model
        +
Moribus system prompt
        +
Structured assessment protocol
        +
Curated behavioural-science knowledge
```

Relevant specialist material may be supplied through retrieval or another controlled knowledge mechanism.

### Level 3 — Model Specialist

```text
Behaviourally specialised model
        +
Moribus system prompt
        +
Structured assessment protocol
```

The underlying model has already been trained, fine-tuned, or otherwise adapted using behavioural or psychological material.

Aestimare should experimentally determine whether increasing specialisation materially improves assessment quality rather than assuming that it does.

---

## Initial Open-Weight Model Candidates

Initial research identified three specialist model families suitable for investigation within the current local resource constraint.

The current experimental ceiling is approximately **14B parameters**.

### 1. Centauri / Qwentaur

**Candidate:** Qwentaur 8B initially  
**Possible later candidate:** Qwentaur 14B  
**Research lineage:** Centaur / Centauri

Centaur and subsequent Centauri research explore models trained or adapted using controlled psychological experimental data.

The Psych-101 dataset used by this research contains behavioural observations from numerous psychology experiments.

Centauri includes models based upon several open-weight foundation-model families, including Qwen.

This makes Qwentaur particularly interesting for Moribus because its specialisation is derived from **experimental behavioural data**, rather than simply general psychological text.

Conceptually:

```text
Controlled behavioural experiments
            |
            v
      Specialist model
            |
            v
Behavioural prediction/reasoning
```

The 8B Qwentaur variant should be investigated first because it remains comfortably within current development resources.

A 14B variant provides a possible later comparison without exceeding the present resource ceiling.

The original approximately 70B Centaur model should remain a research reference but is outside the current practical execution environment.

---

## 2. Be.FM

**Candidate:** BeFM1.5 4B  
**Associated research:** BehaviorBench

BehaviorBench is a behavioural-science benchmark rather than a model.

The associated Be.FM model family is designed specifically around modelling and reasoning about human behaviour.

BehaviorBench includes tasks involving areas such as:

- behavioural prediction;
- behavioural simulation;
- strategic decision-making;
- trait inference;
- behavioural knowledge application.

The **behavioural knowledge application** dimension is particularly relevant to Moribus.

BeFM1.5 4B provides a practical small-model candidate that is sufficiently different from the current Qwen-based development models to provide a useful experimental comparison.

---

## 3. Psyche-R1

**Candidate:** Psyche-R1 7B  
**Foundation:** Qwen2.5

Psyche-R1 is a psychology-specialised model derived from Qwen2.5.

Its specialisation includes psychological questions, reasoning and associated psychological material.

Its original purpose is not identical to Moribus.

This is useful experimentally.

Where Centauri focuses strongly upon experimental behavioural data and Be.FM upon behavioural modelling, Psyche-R1 provides a model whose specialisation is more directly associated with **psychological knowledge and reasoning**.

It therefore provides a different form of domain specialisation against which Moribus can be evaluated.

---

## Control Model

The existing Lumen development model should initially be retained as a control.

**Control:** Qwen2.5-Coder 14B

The control is important precisely because it has **not** been selected or trained specifically for behavioural assessment.

This allows the experiment to distinguish between:

```text
Model specialisation
```

and:

```text
Assessment methodology expressed through prompting
```

If the general coding model performs comparably to specialist behavioural models when supplied with the same Moribus methodology, that is an important result.

It would suggest that specialist model training provides limited additional value for the assessment being tested.

Conversely, systematic improvements from specialist models would provide evidence supporting their use within Moribus.

---

## Initial Experimental Set

The initial experiment should therefore contain four assessors:

| Model | Approximate class | Experimental role |
|---|---:|---|
| Qwen2.5-Coder | 14B | General/coding control |
| Centauri / Qwentaur | 8B | Experimental-behaviour specialist |
| BeFM1.5 | 4B | Behavioural foundation specialist |
| Psyche-R1 | ~7B | Psychology-specialist comparison |

The objective is **not to determine which model is generally best**.

The objective is:

> Determine whether domain-specialised model training materially improves behavioural assessment when the assessment evidence and methodology are held constant.

---

## Controlled Assessment Protocol

Each model should receive the same:

- Aestimare evidence;
- behavioural definitions;
- system-prompt methodology;
- assessment instructions;
- output structure;
- constraints;
- experimental conditions.

A possible Moribus instruction would establish that the assessor must:

> Examine the supplied observations as behavioural evidence. Identify repeatable behavioural effects, relevant changes in stimulus or context, competing explanations, confounding factors, and evidence insufficient to support a conclusion. Characterise observed behaviour only. Do not infer consciousness, emotion, intention, or other unobservable psychological states.

The precise prompt should itself be versioned and treated as part of the experimental evidence.

---

## Experimental Comparison

The first useful experiment should distinguish three effects:

```text
General Model
     +
Basic Assessment Prompt
            |
            v
        Baseline
```

versus:

```text
General Model
     +
Moribus Methodology
            |
            v
    Prompt Specialisation
```

versus:

```text
Specialist Model
     +
Same Moribus Methodology
            |
            v
    Model Specialisation
```

This allows Aestimare to investigate:

1. **Base-model effect**
2. **System-prompt/methodology effect**
3. **Specialist-training effect**

These effects should not be conflated.

---

## Candidate Behavioural Questions

Moribus may eventually assess questions such as:

- Does model behaviour change systematically when a stimulus changes?
- Is behaviour sensitive to framing?
- Does semantically equivalent wording produce materially different behaviour?
- Does prior context systematically influence subsequent behaviour?
- How stable is behaviour under repeated equivalent stimuli?
- Are anchoring-like effects observable?
- How does contradictory context affect subsequent responses?
- How quickly does behaviour adapt following corrective evidence?
- Does behaviour return towards a previous baseline when a stimulus is removed?
- Can observed variation be distinguished from expected probabilistic variation?
- Does an observed behavioural characteristic persist across sessions?
- Does behaviour change following a model, prompt, provider, or orchestration change?

These questions align naturally with Aestimare's experimental and longitudinal evidence model.

---

## Relationship to Consentire

Moribus and Consentire address different problems.

**Moribus asks:**

> What behavioural characteristics are supported by the evidence?

**Consentire asks:**

> Do independent suitably equivalent assessors converge sufficiently upon that assessment?

This permits a future architecture such as:

```text
                    Aestimare Evidence
                           |
                           v
                    Moribus Peritus
                           |
              +------------+------------+
              |            |            |
        Assessor A     Assessor B    Assessor C
              |            |            |
              +------------+------------+
                           |
                           v
                       Consentire
                           |
                           v
             Convergence / Disagreement
                           |
                           v
                 Sufficient Evidence?
```

The models executing Moribus do not necessarily define Moribus.

Instead:

> **Moribus defines the behavioural expertise, methodology, evidence requirements and assessment contract.**

The underlying models provide reasoning capability.

This suggests a useful architectural separation:

```text
Moribus Peritus
    = behavioural expertise and methodology

Model
    = reasoning capability

Praebere
    = model provision

Consentire
    = independent triangulation and convergence

Aestimare
    = evidence and assessment orchestration
```

---

## Aestimare Assessing the Assessor

The apparent recursion created by using Aestimare to evaluate an LLM-based Aestimare assessor is not necessarily a weakness.

It creates an experimentally testable question:

> Is the specialised assessor itself reliable?

Aestimare can accumulate evidence concerning:

- assessor consistency;
- agreement with independent assessors;
- sensitivity to prompt variation;
- sensitivity to evidence ordering;
- reproducibility;
- disagreement patterns;
- systematic biases;
- longitudinal behavioural stability.

This means specialist assessors themselves become observable systems whose behaviour can be characterised.

Consentire can then provide independent assessment convergence where appropriate.

---

## Research Principle

Moribus should begin from an empirical position:

> **Do not assume that a behaviourally specialised model is a better behavioural assessor. Measure whether it is.**

The specialist model is therefore initially a research hypothesis rather than an architectural dependency.

It is entirely possible that:

```text
General model
+
well-designed behavioural methodology
```

performs as well as:

```text
Behaviourally specialised model
+
same methodology
```

That result would be valuable.

Equally, if specialised models demonstrate materially better assessment stability, evidential reasoning, identification of confounding factors, or behavioural characterisation, that provides evidence for incorporating specialised models into Moribus.

---

## Initial Research Plan

1. Identify a small set of existing Lumen traces exhibiting potentially interesting behavioural characteristics.

2. Define and version the first Moribus assessment protocol.

3. Establish Qwen2.5-Coder 14B as the general-model control.

4. Obtain locally runnable versions of:
   - Centauri/Qwentaur 8B;
   - BeFM1.5 4B;
   - Psyche-R1 7B.

5. Confirm execution through Praebere or an experimental equivalent.

6. Present identical evidence and assessment instructions independently to each model.

7. Preserve every assessment and its provenance.

8. Compare:
   - conclusions;
   - evidence selection;
   - behavioural terminology;
   - uncertainty;
   - competing explanations;
   - identification of confounding factors;
   - unsupported inference;
   - reproducibility.

9. Repeat assessments sufficiently to distinguish model behaviour from individual stochastic executions.

10. Use the resulting evidence to determine whether specialist model training materially improves Moribus assessment.

11. Later introduce Consentire to evaluate cross-model convergence and disagreement.

---

## Open Research Questions

- Which behavioural-science concepts transfer usefully to AI model assessment?
- Which concepts become misleading when transferred from human behavioural research?
- How should behavioural terminology be adapted to avoid anthropomorphic claims?
- Does specialist training improve behavioural assessment over prompt specialisation alone?
- Does retrieval of behavioural-science material provide similar benefits without specialist fine-tuning?
- Which specialist model performs most consistently under repeated assessment?
- Does model size materially affect behavioural-assessment quality?
- Does foundation-model diversity improve the evidential value of independent assessment?
- Can Moribus establish behavioural baselines for individual models?
- Can those baselines detect behavioural change following model or orchestration updates?
- Can Aestimare empirically determine task equivalence between candidate Moribus models?
- Can Consentire distinguish genuine independent convergence from correlated model behaviour?

These should remain research questions until supported by experimental evidence.

---

## Working Principle

> **Behaviour can be observed without assuming the nature of the mechanism producing it.**

Moribus applies that principle to AI systems.

Aestimare provides the evidence.

Moribus provides the behavioural methodology.

Praebere provides the models.

Consentire provides independent convergence analysis.

Together they provide a framework in which behavioural claims about AI systems can be tested against preserved, repeatable and longitudinal evidence rather than inferred from isolated responses.