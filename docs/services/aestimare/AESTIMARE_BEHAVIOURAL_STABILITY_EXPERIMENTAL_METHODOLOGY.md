# Lumen Aestimare — Behavioural Stability Experimental Methodology

**Status:** Research / future Aestimare methodology  
**Purpose:** Define an experimental framework for measuring model behaviour under repeated, paraphrased, ambiguous, and contradictory inputs.

## 1. Motivation

A repeated model response can vary for more than one reason. Some variation arises from the probabilistic nature of inference; some is permitted by ambiguity in the input; and some results from the model having to reconcile competing or contradictory signals.

This creates an important distinction for Aestimare:

> **Was the observed divergence introduced by the model, or permitted by the input?**

Aestimare should therefore eventually examine more than whether repeated executions produce the same answer. It should also examine whether an underlying behaviour remains stable when the surface formulation of a task changes.

This methodology is intentionally separate from the initial Aestimare corpus of tasks with definitive answers. The definitive-answer corpus establishes basic evidence and known-answer assessment. The experiments described here form a later behavioural corpus concerned with response stability and sensitivity.

---

## 2. Conceptual Basis

The methodology has similarities to psychometric and trait-analysis techniques used with humans. A latent characteristic is not inferred from a single question. Instead, multiple questions probe the same underlying construct using different wording, framing, ordering, or direction.

The equivalent idea for model assessment is not to claim that a model possesses a human personality or psychological trait. Instead, Aestimare can investigate **behavioural characteristics**, **response dispositions**, and **model behavioural tendencies**.

The central question becomes:

> **Does the model exhibit the same underlying behaviour when the surface formulation changes?**

This is fundamentally different from simply asking whether identical prompts produce identical responses.

---

## 3. Three Distinct Properties

### 3.1 Repeatability

**Definition:** The degree to which repeated executions of the same input produce materially equivalent behaviour or outcomes.

```text
Prompt A -> Run 1
Prompt A -> Run 2
Prompt A -> Run 3
Prompt A -> Run n
```

Replay provides the mechanism for reproducing these executions. Aestimare assesses the significance of any divergence.

Repeatability asks:

> **Given the same task again, does the model behave materially the same way?**

### 3.2 Behavioural Stability

**Definition:** The degree to which a model preserves an underlying behavioural characteristic when the task is expressed differently.

```text
                 Underlying task
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Variant A     Variant B     Variant C
          |            |            |
          +------------+------------+
                       |
                       v
                  Aestimare
```

Behavioural stability asks:

> **Does the same underlying behaviour survive changes in how the problem is presented?**

### 3.3 Semantic Stability

**Definition:** The degree to which the model preserves the relevant meaning, reasoning, decision, or conclusion despite non-material changes to the wording or presentation of the input.

A model may produce different text on every execution while remaining semantically stable. Conversely, a model may produce superficially similar responses while materially changing its interpretation or conclusion.

Semantic stability therefore deliberately separates **textual similarity** from **behavioural equivalence**.

---

## 4. Why the Distinction Matters

A model can be:

- highly repeatable but sensitive to small wording changes;
- textually variable but semantically stable;
- stable under paraphrasing but unstable when irrelevant information is introduced;
- robust to ambiguity until a particular threshold is reached;
- inconsistent when required to reconcile contradictory instructions.

Consequently, response divergence alone is insufficient evidence of poor model behaviour.

If a prompt permits several defensible interpretations, different responses may represent legitimate traversal of the available solution space rather than failure.

This suggests a useful conceptual relationship:

> **Observed response indeterminacy = intrinsic model variability × interpretive freedom × contextual conflict**

This is a conceptual model, not a proposed literal mathematical formula.

---

## 5. Experimental Corpus Structure

Rather than treating every prompt as an independent test, the behavioural corpus should contain **prompt families**.

Each family begins with an underlying task and contains controlled variants of that task.

Example:

```text
Underlying Task T
|
+-- T0: Control / explicit formulation
+-- T1: Semantic paraphrase
+-- T2: Alternative ordering
+-- T3: Irrelevant contextual information
+-- T4: Mild ambiguity
+-- T5: Strong ambiguity
+-- T6: Mild contradiction
+-- T7: Strong contradiction
+-- T8: Positive framing
+-- T9: Negative framing
```

Each variant can itself be executed repeatedly through Replay.

This creates two experimental axes:

```text
                     Surface variation
                           ->
                  T0   T1   T2   T3   T4
                +----+----+----+----+----+
Repeated    R1  |    |    |    |    |    |
execution   R2  |    |    |    |    |    |
     |      R3  |    |    |    |    |    |
     v      Rn  |    |    |    |    |    |
                +----+----+----+----+----+
```

The vertical axis measures repeatability.  
The horizontal axis measures behavioural and semantic stability.

---

## 6. Controlled Experimental Variables

The following variables should initially be manipulated independently wherever practical.

### 6.1 Semantic Paraphrasing

Change wording while preserving the intended meaning and constraints.

Purpose: determine whether behaviour depends materially upon surface language.

### 6.2 Ambiguity

Progressively remove or weaken constraints so that multiple interpretations become increasingly defensible.

Example scale:

```text
A0 -> A1 -> A2 -> A3 -> A4
explicit                  highly ambiguous
```

Purpose: identify how increasing interpretive freedom affects response variance.

### 6.3 Contradiction

Introduce competing evidence, requirements, or instructions with progressively greater conflict.

Example scale:

```text
C0 -> C1 -> C2 -> C3 -> C4
none                     strong conflict
```

Purpose: observe how the model detects, prioritises, reconciles, or fails to reconcile competing signals.

### 6.4 Irrelevant Information

Add context that is factually plausible but unnecessary to solving the task.

Purpose: measure susceptibility to distraction and changes in reasoning caused by non-material context.

### 6.5 Ordering

Change the order in which facts, requirements, evidence, or alternatives are presented without changing their content.

Purpose: identify ordering or recency sensitivity.

### 6.6 Framing

Express the same underlying decision or proposition using different positive, negative, permissive, or restrictive framing.

Purpose: determine whether framing materially changes the model's judgement.

### 6.7 Authority Cues

Where appropriate, introduce claims attributed to an apparent authority while preserving the underlying evidence available to the model.

Purpose: investigate whether unsupported authority signals alter conclusions or evidence weighting.

This category should be introduced carefully and independently of contradiction experiments wherever possible.

---

## 7. Ambiguity and Contradiction Are Different Variables

Ambiguity and contradiction should not be treated as interchangeable.

### Ambiguity expands the solution space.

An ambiguous prompt permits multiple interpretations because information or constraints are missing, weak, or underspecified.

### Contradiction forces reconciliation within the solution space.

A contradictory prompt supplies signals that cannot all be satisfied straightforwardly. The model must recognise, prioritise, reconcile, qualify, or choose between them.

This distinction is important to Aestimare because two divergent responses may have very different causes.

For ambiguity, divergence may be **permitted by the input**.

For contradiction, divergence may expose differences in **conflict resolution behaviour**.

---

## 8. Progressive Disturbance Testing

A particularly useful experiment is to increase a single disturbance incrementally while holding the underlying task constant.

For ambiguity:

```text
A0 -> A1 -> A2 -> A3 -> A4
```

For contradiction:

```text
C0 -> C1 -> C2 -> C3 -> C4
```

Aestimare can then investigate whether there is a point at which behaviour becomes materially unstable.

This introduces the idea of a **behavioural stability threshold**:

> The degree of semantic or contextual disturbance a model tolerates before its behaviour changes materially.

The objective is not necessarily to reduce this to a single universal score. Different task classes may exhibit different thresholds, and the shape of the transition may itself be important evidence.

---

## 9. What Aestimare Should Examine

For each prompt family and repeated execution set, Aestimare should eventually be capable of considering:

- final answer or decision equivalence;
- semantic equivalence;
- reasoning approach;
- evidence selected or ignored;
- assumptions introduced;
- treatment of uncertainty;
- recognition of ambiguity;
- recognition of contradiction;
- conflict-resolution strategy;
- susceptibility to irrelevant context;
- sensitivity to ordering;
- sensitivity to framing;
- tool-use behaviour where applicable;
- capability utilisation;
- materiality of observed divergence.

The purpose remains consistent with the existing Aestimare principle:

> **Aestimare measures behaviour, not intelligence.**

---

## 10. Sufficiency, Correctness, and the Acceptable Solution Space

Many real-world tasks do not have a single canonical correct solution. Software engineering is an obvious example: programming language, deployment target, architecture, service boundaries, persistence technology, coding conventions, and implementation style may all vary while producing solutions that correctly satisfy the underlying objective.

Aestimare should therefore avoid treating difference from one reference outcome as evidence of degradation. A useful distinction is:

- **Correctness:** does the outcome satisfy the necessary requirements?
- **Sufficiency:** is the outcome good enough to achieve the user's objective under the available constraints and bounded resources?
- **Appropriateness:** is the solution well matched to the user's actual circumstances?
- **Optimality:** is the outcome the best available solution according to an explicitly defined objective function?

Without sufficiently defined objectives, constraints, and costs, optimality may not be a meaningful property. A model cannot reliably select *the* optimal architecture, implementation, or decision when the criteria by which optimality is judged are themselves unspecified.

This suggests an important assessment principle:

> **Variation within the space of sufficient solutions is not necessarily behavioural instability.**

A prompt may permit multiple materially different yet valid outcomes:

```text
                         +-- Sufficient solution A
                         +-- Sufficient solution B
Problem + constraints ---+-- Sufficient solution C
                         +-- Sufficient solution D
                         +-- ...
```

Replay may correctly identify divergence between A and B. Aestimare must separately determine whether that divergence matters.

This creates a distinction between **outcome divergence** and **quality degradation**. Two executions may differ substantially in implementation while remaining equivalent with respect to the user's stated objective. Conversely, a superficially similar response may fail an important constraint and therefore represent meaningful degradation.

### 10.1 Sufficiency Is Bounded by Resources

Sufficiency is inseparable from bounded resources. Relevant bounds may include:

- time available;
- computational resources;
- financial cost;
- existing skills and organisational knowledge;
- deployment environment;
- maintenance burden;
- operational complexity;
- risk tolerance;
- required performance;
- available evidence;
- acceptable decision latency.

The same solution can therefore be sufficient under one set of bounds and inappropriate under another. A small Python service may be sufficient for one workload, while a different implementation becomes appropriate when throughput, deployment, team skills, or delivery constraints change.

This is consistent with bounded rationality and satisficing: the cost of continuing to search for a theoretically superior solution is itself part of the decision problem.

### 10.2 Simplicity Is Contextual

Simplicity should not be treated as a single universal property. A solution may be simple to write but difficult to deploy; simple to deploy but expensive to operate; architecturally simple but organisationally difficult to maintain.

Aestimare should therefore treat claims of simplicity relative to a dimension and stakeholder:

> **Simple for whom, in what dimension, and over what period?**

This matters when comparing model outputs. A shorter or less elaborate answer is not necessarily evidence of lower capability. It may represent better judgement about the amount of capability required to satisfy the task.

### 10.3 Capability Utilisation and Sufficiency

Maximum capability utilisation should not automatically be interpreted as higher-quality behaviour. If a task requires only a small portion of the model's available capability, using substantially more reasoning, tooling, context, or implementation complexity may constitute unnecessary expenditure rather than improvement.

This suggests a useful conceptual progression:

```text
Possible -> Correct -> Sufficient -> Appropriate -> Optimal
```

Aestimare should eventually be able to distinguish where an outcome lies within this progression rather than assuming that every task requires a singular maximally optimised answer.

### 10.4 Implication for Divergence Assessment

When Replay detects divergence, Aestimare should eventually be capable of asking:

1. Did the executions differ?
2. Is the difference semantic or merely surface-level?
3. Do both outcomes satisfy the explicit requirements?
4. Do both remain sufficient under the user's stated bounded resources?
5. Is one outcome materially more appropriate under those bounds?
6. Was the divergence permitted by a legitimate solution space, or does it represent behavioural degradation?

This extends the earlier distinction between **model-originated divergence** and **input-permitted divergence**. Some divergence is not merely permitted by ambiguity; it is inherent in tasks for which multiple valid solutions exist even when the requirements are clear.

For such tasks, the assessment target should not be exact solution convergence. It should be **stability of quality, constraint satisfaction, and sufficiency across a legitimate solution space**.

---

## 11. Divergence Classification

The experimental framework suggests that Aestimare should eventually distinguish at least the following causes or interpretations of divergence:

### Model-originated divergence

The input remains sufficiently constrained, but materially different behaviour occurs across otherwise comparable executions.

### Input-permitted divergence

The input admits multiple defensible interpretations, and executions select different interpretations.

### Conflict-resolution divergence

The input contains competing signals and executions resolve those signals differently.

### Surface divergence

The wording or structure of the response changes while the underlying behaviour and outcome remain materially equivalent.

### Semantic divergence

The underlying interpretation, reasoning, decision, or conclusion changes materially.

These classifications should initially be treated as assessment concepts rather than hard-coded implementation requirements.

---

## 12. Relationship to Lumen Services

The methodology fits naturally into the existing Lumen service responsibilities.

**Vestigare / Trace** captures the evidence of what occurred.

**Repetere / Replay** reproduces executions and exposes divergence without judging it.

**Aestimare / Assess** determines whether the observed differences are meaningful and characterises the behaviour represented by them.

**Fiducia** may later automate experimental execution, scheduling, aggregation, and longitudinal analysis.

The separation remains important:

> Replay reproduces. Aestimare judges.

Aestimare may therefore filter, aggregate, compare, or reinterpret evidence produced across many Replay executions without requiring Replay itself to understand the experimental meaning of those executions.

---


## 13. Aestimare as an Expert-Routed Assessment Service

As the range of assessment responsibilities grows, Aestimare should retain a single stable external contract while separating specialised assessment capabilities internally. The objective is not to expose each assessment mechanism as another top-level Lumen service, but to allow assessment engines to mature independently behind the Aestimare boundary.

Conceptually, this resembles a **Mixture of Experts (MoE)** architecture:

```text
Evidence / Assessment Request
            |
            v
       Aestimare API
            |
            v
   Applicability / Routing Policy
            |
    +-------+--------+---------+
    |       |        |         |
    v       v        v         v
Correctness  Sufficiency  Behavioural  Semantic
  Engine       Engine       Stability    Stability
                            Engine       Engine
    |       |        |         |
    +-------+--------+---------+
            |
            v
    Assessment Aggregation
            |
            v
 Consolidated Aestimare Result
```

The MoE analogy is useful but should not be used as the formal architectural description without qualification. Conventional neural Mixture-of-Experts systems route tokens among internal model components. Aestimare instead routes explicit evidence and assessment requests among **inspectable assessment modules with defined responsibilities, evidence requirements, and outputs**.

A more precise description is therefore:

> **Aestimare is an expert-routed assessment service.**

### 13.1 Stable External Contract

To the rest of Lumen, the responsibility remains deliberately simple:

> **Aestimare assesses.**

Vestigare, Repetere, Fiducia, Rogare, or other consumers should not need to know which internal assessment engine is appropriate. They submit evidence and assessment context to Aestimare and receive an assessment result.

This preserves service-level simplicity while allowing the assessment subsystem to become sophisticated internally.

### 13.2 Specialised Assessment Engines

Initial internal domains may include, but are not limited to:

- **Correctness assessment** — whether an outcome satisfies independently verifiable requirements or known answers.
- **Sufficiency assessment** — whether an outcome is good enough for the stated objective under bounded resources and constraints.
- **Behavioural stability assessment** — whether behaviour remains materially stable across repeated executions or controlled disturbances.
- **Semantic stability assessment** — whether meaning, reasoning, or conclusions remain materially equivalent despite surface variation.
- **Conflict and ambiguity assessment** — how ambiguity is recognised and how competing or contradictory signals are reconciled.
- **Capability-utilisation assessment** — whether additional reasoning, tooling, context, or complexity materially improves the outcome.
- **Environmental invariance assessment** — whether explicitly defined model execution conditions remained sufficiently stable and immutable for behavioural comparisons to be treated as experimentally comparable.

These are internal assessment responsibilities, not necessarily separate deployable services. Their boundaries should nevertheless be explicit enough that an engine can evolve, be replaced, or eventually be separated without changing Aestimare's external contract.

### 13.3 One Job First — Micro-Engine Responsibility

The internal assessment architecture should apply Lumen's **one job first** principle as rigidly to each assessment engine as it does to the surrounding services.

> **Each Aestimare micro-engine should answer one narrowly defined assessment question.**

An engine should consume the evidence relevant to that question and return a specialist assessment. It should not expand its responsibility merely because adjacent evidence is available or because it could conveniently make another judgement.

For example:

- an **environmental invariance engine** determines whether the explicitly defined model execution conditions remained sufficiently stable and immutable across an experiment; it does not decide whether behavioural divergence is acceptable;
- a **behavioural divergence engine** establishes and characterises material behavioural difference; it does not infer why that divergence occurred;
- a **sufficiency engine** determines whether an outcome remains sufficient for the stated objective under the defined bounds; it does not determine whether the experimental environment was controlled;
- a **semantic stability engine** determines whether relevant meaning, reasoning, decision, or conclusion remains materially equivalent; it does not assess every other dimension of response quality.

This produces a deliberate hierarchy of responsibility:

```text
Evidence services observe.
        |
        v
Micro-engines assess one thing.
        |
        v
Aestimare synthesises assessments.
```

Aestimare itself therefore also retains one primary job: **coordinate applicable specialist assessments and construct the consolidated assessment without absorbing their specialist responsibilities.**

A useful design test follows:

> **If an engine begins assessing a second independent property because "we already have the evidence", that property should normally become another engine.**

This constraint is central to modularity. New evidence sources, research methods, or third-party capabilities should normally result in a new specialist assessor where a genuinely new assessment question exists, rather than additional conditional branches accumulating inside an existing assessor.

The micro-engine model also prevents external integrations from defining Aestimare's architecture. A third-party capability may provide environmental evidence, linguistic evidence, or some future form of specialised evidence, but Aestimare remains bound to its own assessment contracts. The external implementation is an evidence source or capability provider; it is not the definition of the assessment engine.

Real integrations can therefore help **discover** useful abstractions without Lumen becoming dependent upon those implementations.

> **Use real integrations to discover the abstraction; do not make the abstraction specific to the integration.**

This preserves the stable Aestimare boundary while allowing the internal assessment system to grow incrementally and remain comprehensible as new forms of evidence and assessment are introduced.

### 13.3 Applicability and Routing

Not every assessment engine should run for every task. Aestimare should determine which forms of assessment are applicable from the request, task characteristics, experiment definition, and available evidence.

For example:

- a deterministic arithmetic task may primarily require correctness assessment;
- a repeated identical prompt may require repeatability and behavioural stability assessment;
- a prompt-family experiment may require semantic and behavioural stability assessment;
- a software design task may require correctness, constraint satisfaction, sufficiency, simplicity, and capability-utilisation assessment;
- a contradiction experiment may require explicit conflict-recognition and reconciliation analysis.

The routing decision is itself part of the assessment provenance and should therefore be observable. Aestimare should eventually be able to answer:

- Why was this engine selected?
- What evidence did it consume?
- What did it conclude?
- With what confidence or limitations?
- How did its result contribute to the consolidated assessment?

### 13.4 Aggregation Is Not Forced Consensus

Multiple assessment engines may produce findings that are complementary, orthogonal, or in tension. The aggregation layer should not simply average scores or force every engine into a single scalar result.

For example, an execution may be:

- correct but unnecessarily complex;
- semantically divergent but equally sufficient;
- repeatable but highly sensitive to paraphrasing;
- insufficient despite being textually similar to a reference response.

The consolidated assessment should preserve these distinctions rather than erase them. Aestimare's role is to construct an interpretable judgement from specialised evidence, not manufacture artificial agreement.

### 13.5 Extensibility and Third-Party Assessment

The expert-routed structure also provides a natural integration point for future assessment methodologies. A new internally developed method, research-derived technique, or third-party capability may provide evidence that is consumed by an existing specialist engine, or may justify a new micro-engine when it introduces a genuinely distinct assessment question.

The distinction is important: third-party implementations should not define Aestimare's internal contracts. They provide capabilities or evidence through Lumen integration boundaries; Aestimare remains responsible for interpreting that evidence through narrowly scoped assessment engines.

This avoids promoting every useful mechanism to a first-class Lumen service or binding Aestimare to a particular external implementation. Aestimare becomes the stable boundary through which **assessment expertise** is selected, invoked, traced, and combined.

The architectural principle is therefore:

> **Keep the external contract as Aestimare, but internally separate assessment engines so they can mature independently.**

---

## 14. Experimental Progression

This research should be introduced incrementally.

### Phase 1 — Definitive-answer baseline

Use tasks with clear, independently verifiable answers. Establish the mechanics of repeated execution, evidence collection, divergence detection, and assessment.

### Phase 2 — Paraphrase stability

Create semantically equivalent prompt families and measure whether underlying behaviour remains stable across wording changes.

### Phase 3 — Controlled ambiguity

Introduce progressively ambiguous variants and observe changes in interpretation and response variance.

### Phase 4 — Controlled contradiction

Introduce progressively conflicting evidence or instructions and examine conflict-resolution behaviour.

### Phase 5 — Multi-variable behavioural corpus

Add ordering, framing, irrelevant information, authority cues, and other controlled disturbances.

### Phase 6 — Longitudinal characterisation

Repeat selected experimental families across models, model versions, system prompts, tool configurations, and time.

This enables Aestimare to move from evaluating individual executions toward characterising persistent behavioural tendencies.

---

## 15. Research Questions

The methodology creates a number of concrete research questions for future Aestimare work:

1. How repeatable is a model when presented with an identical task?
2. How stable is its behaviour across semantically equivalent formulations?
3. At what degree of ambiguity does material behavioural variance increase?
4. At what degree of contradiction does conflict-resolution behaviour become unstable?
5. Does the model recognise ambiguity before resolving it?
6. Does the model explicitly recognise contradictory evidence or silently choose between it?
7. How sensitive is behaviour to irrelevant context?
8. How sensitive is behaviour to information ordering?
9. Does framing alter conclusions when the underlying evidence is unchanged?
10. Are observed behavioural tendencies stable across repeated experiments?
11. Do those tendencies change between model versions or system configurations?
12. Does tool availability alter behavioural stability even when the underlying task is unchanged?
13. Can Aestimare distinguish meaningful semantic divergence from harmless textual variation?
14. Can a behavioural stability threshold be characterised for particular task classes?
15. Can Aestimare distinguish divergence within a legitimate solution space from quality degradation?
16. Are different solutions equally sufficient under the user's bounded resources?
17. Does increased capability utilisation improve the outcome, or merely increase complexity and cost?
18. Can simplicity be assessed relative to the dimension that matters to the user rather than as an absolute property?
19. Can Aestimare reliably select the appropriate assessment engines from task characteristics and available evidence?
20. How should findings from multiple assessment engines be consolidated without obscuring disagreement or orthogonal dimensions?
21. Can new or third-party assessment engines be introduced without changing Aestimare's external contract?

---

## 16. Longer-Term Significance

Repeated identical prompts tell us something useful about nondeterminism, but they provide only one view of model behaviour.

A more consequential question is whether behaviour survives changes that should not materially alter the underlying task.

This gives Lumen three complementary concepts:

> **Repeatability:** Does the model behave similarly when we repeat the same execution?

> **Behavioural stability:** Does the underlying behaviour persist when we alter how the task is presented?

> **Semantic stability:** Does meaning remain materially equivalent despite textual or contextual variation?

> **Sufficiency stability:** Do materially different outcomes remain within the acceptable solution space and continue to satisfy the objective under the same bounded resources?

Together, these allow Aestimare to move beyond observing that probabilistic systems diverge and toward experimentally characterising **when, how, and why their behaviour changes**.

That is the purpose of this proposed behavioural stability corpus.
