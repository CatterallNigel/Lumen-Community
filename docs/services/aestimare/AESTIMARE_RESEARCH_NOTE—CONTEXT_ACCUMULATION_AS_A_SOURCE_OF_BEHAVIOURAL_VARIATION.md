# Aestimare Research Note — Context Accumulation as a Source of Behavioural Variation

## Purpose

This note defines a research area for **Lumen Aestimare (Assess)** concerning behavioural changes caused by the accumulation of conversational context.

The central proposition is:

> **A model's behaviour may change during a long-running session not because the model has failed, but because the accumulated context has changed the effective problem the model is solving.**

To the user, this change may appear inexplicable. The model may appear to have "lost the plot", become inconsistent, or started interpreting otherwise straightforward requests incorrectly.

However, the model may actually be responding consistently to information contained within the complete conversational context.

The problem is therefore not necessarily loss of context.

It may be **divergence between the context effectively being used by the human and the context available to the model**.

---

## 1. The Initial Session

At the beginning of a session, excluding system and assistant instructions, the conversational state is comparatively simple:

```text
User Prompt
    ↓
Model Interpretation
    ↓
Response
```

Any ambiguity or contradiction is predominantly contained within the user's prompt.

In many cases this ambiguity is intentional: resolving what the user means is itself part of the task.

For example:

```text
User:
"Which implementation would be better here?"
```

The model may need to infer what "better" means from the supplied requirements.

This is ordinary prompt interpretation.

---

## 2. Context Changes the Problem

As the conversation progresses, each interaction adds information to the model's available context.

This may include:

- facts;
- assumptions;
- terminology;
- decisions;
- rejected alternatives;
- qualifications;
- corrections;
- temporary hypotheses;
- unresolved questions;
- changes in direction;
- inferred user intent;
- previous model interpretations.

The model is therefore no longer responding solely to the current user request.

Conceptually:

```text
Current User Request
        +
Accumulated Conversational State
        ↓
Model Interpretation
        ↓
Response
```

The accumulated state can materially affect how the current request is interpreted.

---

## 3. Human and Model Effective Context

A significant asymmetry exists between the human participant and the model.

A model may have access to tens of thousands of tokens from the conversation.

The human participant does not normally reconsider that entire conversation before making the next request.

The human's effective working context may instead resemble:

```text
Current Objective
       +
Recent Discussion
       +
Previous Model Response
       ↓
Next User Request
```

The model's effective context may resemble:

```text
Turn 1
Turn 2
Turn 3
...
Turn 95
       +
Turn 96 User Request
       ↓
Next Response
```

This creates the possibility that the human and model are responding to **different effective conversational states while participating in the same conversation**.

---

## 4. Contextual Contradiction Can Be Emergent

An important distinction is that a contradiction does not have to exist within an individual prompt.

It may emerge only when statements made at different points in the conversation are considered together.

For example:

```text
Turn 10:
"Replay should reproduce the original execution exactly."

Turn 70:
"Replay should become transparent once divergence occurs."

Turn 130:
"After divergence, why didn't Replay continue enforcing the original execution?"
```

Each statement is individually reasonable.

Taken together, however, they potentially contain incompatible expectations.

The user at Turn 130 may primarily be reasoning from the immediately preceding discussion.

The model may still be incorporating the decisions established at Turns 10 and 70.

The resulting response can therefore appear incorrect or confused to the user while remaining explicable from the model's accumulated context.

---

## 5. "The Model Has Lost the Plot"

A common user experience in long conversations is the perception that:

> "The model has lost the plot."

This observation is behaviourally important.

The user has detected a change in the model's responses.

However, the cause of that change is presently opaque.

Possible explanations include:

- model instability;
- ambiguity in the latest request;
- accumulated ambiguity;
- contradictory earlier instructions;
- superseded decisions remaining influential;
- incorrect model inference;
- excessive irrelevant context;
- interaction between recent and historical instructions;
- divergence between human and model effective context.

Without provenance and contextual analysis, these explanations are difficult to distinguish.

The observed behavioural change is therefore real, while its cause remains unexplained.

---

## 6. More Context Is Not Necessarily Better Context

Large context windows are generally presented as an increase in model capability.

That is true in the sense that additional information can remain available to the model.

However:

> **More available context does not necessarily mean better effective context.**

A conversation may move through several stages:

```text
Insufficient Context
        ↓
Useful Context
        ↓
Sufficient Context
        ↓
Redundant Context
        ↓
Ambiguous / Conflicting Context
        ↓
Behavioural Degradation
```

This suggests that context utility may not increase monotonically with context size.

There may instead be a point at which additional conversational history begins to introduce greater interpretive pressure than informational value.

This is especially important because the problem can occur long before the model's technical context-window limit is reached.

The relevant limit may therefore not be:

> How much context can the model accept?

but:

> **How much context can remain behaviourally useful before accumulated state begins materially affecting reasoning quality?**

---

## 7. Context as Behavioural Pressure

Aestimare should therefore consider context not merely as information supplied to the model, but as an **independent source of behavioural pressure**.

A model response is potentially influenced by:

```text
Model
+
System Instructions
+
Assistant Instructions
+
Current Prompt
+
Accumulated Context
+
Tools
+
External Information
+
Execution Conditions
```

Consequently, two apparently equivalent requests may produce different behaviour depending upon the conversational state in which they occur.

This makes accumulated context a candidate explanatory variable for behavioural variation.

---

## 8. Aestimare Research Question

The primary research question is:

> **To what extent does accumulated conversational context cause measurable behavioural variation in otherwise comparable model requests?**

Supporting questions include:

1. Does behavioural divergence increase as conversational context grows?

2. Does ambiguity accumulation correlate with behavioural divergence?

3. Does explicit contradiction produce greater behavioural variation than irrelevant context of equivalent size?

4. Do superseded decisions continue to influence later responses?

5. How strongly does recent context dominate historical context?

6. Can the point at which context becomes behaviourally detrimental be detected?

7. Can behavioural changes attributed to context be distinguished from ordinary model stochasticity?

8. Does the same model exhibit different sensitivity to different forms of contextual conflict?

9. Can a model remain internally consistent with accumulated context while appearing inconsistent to the human participant?

10. Can contextual provenance explain a behavioural divergence after it occurs?

---

## 9. Candidate Experimental Classes

Aestimare should eventually support controlled experiments covering several context conditions.

### Clean Context

A request executed with only the minimum required context.

This provides the baseline.

### Increasing Relevant Context

The same task executed after progressively increasing amounts of relevant conversational history.

This examines whether additional relevant information improves or alters behaviour.

### Increasing Irrelevant Context

The same task executed with increasing quantities of unrelated but valid conversational history.

This tests whether context volume alone produces behavioural change.

### Ambiguous Context

Earlier conversation introduces statements capable of supporting multiple interpretations of the later request.

### Contradictory Context

Earlier statements establish mutually incompatible expectations.

### Superseded Context

An earlier decision is explicitly replaced later in the conversation.

This tests whether the model correctly prioritises the newer state.

### Temporally Separated Contradiction

Contradictory statements are deliberately separated by substantial conversational distance.

This tests whether contextual position influences behaviour.

### Human-Context Simulation

The final request is constructed naturally from only the immediately preceding discussion while historical context contains information capable of changing its interpretation.

This approximates the real-world condition in which the human's effective context is substantially shorter than the model's.

---

## 10. Experimental Structure

A useful experimental pattern would be:

```text
Baseline Execution
    ↓
Introduce Controlled Context
    ↓
Repeat Equivalent Task
    ↓
Trace Execution
    ↓
Replay Where Appropriate
    ↓
Assess Behavioural Difference
```

The important property is that the final task remains as constant as practical while the preceding context is manipulated.

This allows context itself to become the experimental variable.

---

## 11. What Aestimare Should Measure

Potential measurements include:

- answer divergence;
- interpretation divergence;
- decision divergence;
- reasoning-path divergence;
- tool-selection divergence;
- capability-utilisation changes;
- instruction-priority changes;
- factual consistency;
- adherence to established decisions;
- adherence to superseding decisions;
- confidence changes;
- clarification behaviour;
- response length and structure;
- references to historical context;
- references to recent context.

Not every difference represents degradation.

Aestimare must distinguish:

```text
Behaviour Changed
```

from:

```text
Behaviour Became Worse
```

This remains consistent with the wider Aestimare principle:

> **Assess measures behaviour, not intelligence.**

---

## 12. Provenance and Explanation

This research area also demonstrates why behavioural provenance matters.

Observability can establish:

```text
Response behaviour changed at Turn 96.
```

Reasoning assurance should attempt to establish:

```text
Behavioural divergence increased after Turn 58,
where an assumption was introduced that conflicts
with the decision established at Turn 12.

Subsequent responses remained compatible with the
accumulated conversational state but increasingly
diverged from the user's recent conversational direction.
```

The second explanation is considerably more useful.

It transforms an apparently inexplicable model failure into an inspectable relationship between context and behaviour.

---

## 13. Relationship to Lumen

This problem naturally crosses several Lumen services.

**Vestigare (Trace)** preserves the execution evidence and contextual provenance.

**Repetere (Replay)** enables comparable executions to be reproduced under controlled conditions.

**Aestimare (Assess)** evaluates whether changing context produced meaningful behavioural variation.

**Fiducia** may eventually automate repeated experiments and identify statistically meaningful behavioural patterns.

**Moderari** is particularly relevant because system and orchestration context must be controlled or accounted for when experiments are performed.

The objective is not for these services to independently determine "the correct answer."

Their combined purpose is to make behavioural change observable, reproducible, measurable and explainable.

---

## 14. Relationship to Bounded Rationality

This research also intersects with Lumen's existing work around bounded resources and satisficing.

Context-window size describes a technical capacity.

It does not establish that every item contained within that window remains equally useful to the reasoning process.

A larger context window increases the quantity of information available while potentially increasing:

- ambiguity;
- conflicting constraints;
- obsolete information;
- competing interpretations;
- irrelevant information;
- dependency on historical assumptions.

The relevant question therefore becomes one of **effective contextual utility**, rather than maximum contextual capacity.

This parallels the broader distinction between capability and useful utilisation of capability.

---

## 15. Working Hypothesis

The initial Aestimare hypothesis should be:

> **As conversational context accumulates, behavioural variation can emerge before any technical context limit is reached. This variation may be caused by ambiguity, contradiction, obsolete state or divergence between human and model effective context.**

A secondary hypothesis is:

> **Some behaviour perceived by users as the model "losing the plot" will prove to be rationally explainable from the accumulated machine-visible context.**

If demonstrated experimentally, this distinction would be significant.

The model may not have forgotten the conversation.

**It may, in some cases, remember more of the conversation than the human does.**

The resulting behavioural problem is therefore not simply memory failure.

It is a **context alignment problem between human and machine**.

---

## 16. Research Principle

This suggests an additional principle for Aestimare:

> **Context is not merely input. Context is an experimental variable.**

And, more specifically:

> **Behavioural assurance requires understanding not only what context a model had, but which parts of that context materially influenced its behaviour.**

That distinction should form part of Aestimare's future corpus and experimental methodology.