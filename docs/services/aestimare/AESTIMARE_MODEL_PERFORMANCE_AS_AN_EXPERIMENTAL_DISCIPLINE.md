# Aestimare — Model Performance as an Experimental Discipline

## Status

Exploratory architecture and research concept.

This document describes a conceptual model for **Lumen Aestimare**: treating the assessment of trained AI models in a manner analogous to modern sports performance analysis.

The analogy is not intended to suggest that models and people are equivalent. Its value is methodological. Modern sport distinguishes an athlete's underlying capability from their observed performance and uses repeated, instrumented evidence to understand the conditions under which that performance changes.

Aestimare applies a similar principle to trained models.

> **Assess measures behaviour, not intelligence.**

---

## 1. The Trained Model as the Athlete

A trained model can be considered analogous to an already-trained athlete.

The model's training has established capabilities, limitations and behavioural characteristics. Aestimare is not primarily concerned with reconstructing how those capabilities arose inside the neural network.

Instead, it asks what the trained model actually does when required to perform.

This creates a useful separation:

- **training** creates or changes underlying model capability;
- **execution** exposes observable model behaviour;
- **assessment** characterises that behaviour from evidence;
- **experimentation** changes conditions and observes whether behaviour changes.

The equivalent question in sport is not simply:

> How was this athlete trained?

It is also:

> What can this athlete actually do, under which conditions, how consistently, and what changes their performance?

That is the territory Aestimare is intended to investigate for models.

---

## 2. Capability Is Not Performance

An athlete's underlying capability and their realised performance are not identical.

Performance can be influenced by:

- the event being attempted;
- instructions and strategy;
- preparation;
- environment;
- equipment;
- fatigue;
- psychological state;
- opposition;
- repeated workload; and
- other conditions surrounding the performance.

Similarly, a trained model's weights can remain unchanged while its observable behaviour changes materially.

Relevant conditions can include:

- task type;
- system prompt;
- user prompt;
- available context;
- preceding conversation;
- tool availability;
- model parameters;
- orchestration;
- provider configuration; and
- execution environment.

A useful distinction for Aestimare is therefore:

> **Model capability ≠ observed model behaviour ≠ realised task performance.**

A model may possess the capability required for a task without that capability being elicited reliably under every set of conditions.

Conversely, changing the conditions around the same trained model may materially improve or degrade its realised performance without changing the model itself.

---

## 3. From Observation to Experimental Evidence

A single sporting performance provides evidence, but it does not completely characterise an athlete.

The same principle applies to model executions.

A single successful or unsuccessful execution is an observation. Repetition under controlled conditions allows consistency and variation to become visible. Changing one or more conditions creates an experiment. Repeating experiments over time begins to establish behavioural characteristics.

Aestimare therefore adopts the progression:

> **A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows us to detect behavioural change.**

This is important because probabilistic model behaviour cannot be adequately characterised from isolated examples.

A benchmark result, demonstration or single successful execution can establish that something happened.

It cannot by itself establish how reliably it happens, under which conditions it continues to happen, or what causes the behaviour to change.

---

## 4. The Rise of Performance Analytics

Modern elite sport increasingly relies upon data rather than observation alone.

Athletes and teams are instrumented and analysed through:

- timing;
- workload;
- biomechanics;
- environmental conditions;
- recovery;
- repeated trials;
- historical performance;
- comparative statistics;
- intervention analysis; and
- longitudinal trends.

The athlete has not become a fundamentally different biological object because these measurements exist.

What has changed is the ability to **observe, characterise and optimise performance**.

Aestimare seeks a similar progression for trained AI models.

Rather than treating each model response as an isolated answer, Lumen can preserve evidence about the execution and the conditions surrounding it. Repetition and comparison can then turn individual executions into a body of behavioural evidence.

The objective is not merely to collect more data.

The objective is to make claims about model performance increasingly evidence-based.

---

## 5. A Sports Performance Mapping

The analogy can be expressed as follows:

| Sports Performance | Aestimare |
|---|---|
| Athlete | Trained model |
| Sporting event | Task or task class |
| Performance | Model execution |
| Timing and telemetry | Trace evidence |
| Repeated trials | Replay experiments |
| Conditions | Context, prompts, parameters and tools |
| Coaching/instructions | System and user prompting |
| Strategy | Orchestration |
| Performance statistics | Aestimare measurements |
| Specialist analysts | Periti Assessores |
| Resource/economic analysis | Oeconomia |
| Personal best | Observed capability frontier |
| Consistency | Behavioural stability |
| Performance degradation | Behavioural change or drift |
| Different disciplines | Different task classes |
| Longitudinal record | Evidence accumulated over time |

The mapping is deliberately conceptual rather than literal.

Its value is that it prevents several common assumptions.

A single performance does not define an athlete.

A single execution does not define a model.

An athlete who excels in one discipline is not necessarily the best athlete for another.

A model that excels on one benchmark or task class is not necessarily the best model for another.

And maximum theoretical capability does not necessarily represent the most appropriate performance for a particular requirement.

---

## 6. Fitness for Purpose Rather Than Universal Ranking

The question:

> Which athlete is best?

is largely meaningless without defining the event and the conditions.

A world-class marathon runner and a world-class sprinter possess different performance characteristics. Ranking one universally above the other discards the purpose for which their capabilities are being evaluated.

The same problem exists with models.

Aestimare should resist reducing model assessment to a universal score such as:

> Model A: 93  
> Model B: 87  
> Therefore Model A is better.

A more useful assessment is contextual:

> For task class X, under conditions Y, Model A repeatedly exhibits these characteristics.

and:

> For task class Z, under conditions Q, Model B repeatedly exhibits these characteristics.

The relevant question becomes:

> **Which model demonstrates the required characteristics for this task under these conditions?**

This is **fitness for purpose**, rather than abstract model superiority.

---

## 7. Conditions as Performance Variables

One of the most important consequences of this approach is that the conditions surrounding execution become part of the evidence.

Conceptually:

```text
Model
  + Task
  + System Prompt
  + User Prompt
  + Context
  + Tools
  + Parameters
  + Orchestration
        |
        v
    Execution
        |
        v
     Evidence
        |
        v
 Periti Assessores
        |
        v
 Characterised Performance
        |
        v
  Compare / Repeat
        |
        v
 Evidence of what works
```

This makes it possible to investigate not merely whether a model succeeded, but what changed when its performance changed.

For example:

- Did a different system prompt materially affect behaviour?
- Did additional context improve task performance?
- At what point did additional context stop helping?
- Did tool availability change the quality or consistency of the result?
- Did orchestration alter the model's ability to complete the task?
- Does the same configuration repeatedly produce comparable behaviour?
- Does a model require more explicit instruction than another model for the same task?

These are experimental questions.

Aestimare provides the architectural location in which evidence relevant to those questions can be assessed.

---

## 8. The Sports Psychology Analogy

Sports psychology provides another useful conceptual parallel.

An athlete may possess the physical capability required to perform, yet their realised performance can still be affected by instruction, preparation, confidence, framing, pressure and strategy.

The intervention does not necessarily change the athlete's underlying physical capability.

It changes the conditions under which that capability is expressed.

Something comparable can occur with a trained model.

Changing:

- system instructions;
- task framing;
- contextual information;
- ordering of information;
- conversational history;
- tool access; or
- orchestration

can change observable performance while the model weights remain identical.

Aestimare is therefore interested not only in **what capability exists**, but in **what conditions elicit, suppress or alter observable performance**.

This does not require a claim about model cognition or human-like psychology.

The analogy concerns experimentally observable performance effects.

---

## 9. Optimisation Without Retraining

This distinction creates an important area of investigation.

Improving model performance does not necessarily require retraining the model.

If repeated evidence demonstrates that a particular configuration, context strategy, prompt structure, tool arrangement or orchestration pattern produces better results for a task class, then the realised performance of the existing trained model may be improved operationally.

This resembles performance optimisation in sport.

The athlete remains the same athlete, but evidence can inform changes to preparation, strategy, equipment or coaching.

Likewise, the model remains the same trained model while the system around it is adjusted based upon observed evidence.

Aestimare can therefore contribute to understanding:

> **How do we obtain better, more reliable or more appropriate performance from the capability that already exists?**

This is distinct from improving the underlying model through training.

---

## 10. The Role of the Periti Assessores

Modern sports performance analysis is multidisciplinary.

Different specialists examine different aspects of the same athlete and performance. No single measurement completely describes performance.

Aestimare follows a similar architectural principle through the **Periti Assessores**.

Each Peritus examines a distinct assessment question using evidence and methodology appropriate to that domain.

For example:

- **Oeconomia** examines economic efficiency and the resources required to obtain and consume satisfactory outcomes;
- behavioural stability may require a different specialist assessment;
- context sensitivity may require another;
- task sufficiency may require another;
- other Periti should emerge only when a distinct assessment question has been identified.

The same execution can therefore be examined from several independent perspectives without collapsing those perspectives into one artificial score.

Aestimare coordinates these specialised findings into a broader assessment.

---

## 11. Oeconomia in the Performance Model

The sports analogy also helps explain why economic assessment belongs within Aestimare.

The most capable athlete is not necessarily the economically appropriate athlete for every task.

Likewise, using the most capable frontier model for every execution may provide little additional utility while imposing substantially greater cost.

Oeconomia asks a different question from a capability benchmark:

> **What resources were required to obtain a satisfactory performance?**

Those resources can include machine expenditure and human interaction burden.

The economically appropriate model therefore depends upon the work being performed.

A small model that reliably performs a simple task may represent better fitness for purpose than a much more capable and expensive model.

For a difficult task, the opposite may be true if the more capable model reduces prompting, retries, verification or failure.

Performance and economics must therefore be considered together without treating them as the same measurement.

---

## 12. Longitudinal Performance

Sports analytics becomes particularly valuable when evidence is accumulated over time.

A single result can show performance.

A longitudinal record can show:

- improvement;
- degradation;
- stability;
- variability;
- response to interventions;
- changes under different conditions; and
- whether previously observed characteristics remain true.

The same is potentially true for models.

A model provider may update a model or serving environment. A system prompt may evolve. Tooling may change. Context-management strategies may change. Or repeated observations may simply reveal variability that was invisible in a smaller sample.

Aestimare should therefore treat historical evidence as more than an archive.

It provides the basis for asking:

> **Has the observed behaviour changed?**

and, where the evidence permits:

> **What changed around the execution when the behaviour changed?**

This is one of the reasons behavioural evidence becomes more valuable as it accumulates.

---

## 13. What Aestimare Is Not

The performance-analysis model also helps establish boundaries.

Aestimare is not primarily:

- a neural-network interpretability system;
- a model-training framework;
- a universal intelligence benchmark;
- a leaderboard;
- a mechanism for declaring one model universally superior;
- or a system that infers broad behavioural conclusions from isolated executions.

It is concerned with observable performance and the evidence surrounding that performance.

The aim is not to explain every internal mechanism responsible for a model's behaviour.

The aim is to characterise what can be observed sufficiently well that decisions about model use can be based upon evidence rather than assumption, reputation or isolated anecdote.

---

## 14. Core Principle

The sports-performance analogy can be reduced to a simple distinction:

> **Training establishes capability. Performance reveals how that capability is expressed under particular conditions.**

For Aestimare:

> **Aestimare does not ask how the model was made. It asks how the trained model performs, under what conditions, how reliably, at what cost, and what changes that performance.**

This is why Aestimare measures behaviour rather than intelligence.

Its purpose is to turn executions into observations, observations into experiments, experiments into characterised behaviour, and accumulated evidence into a more reliable understanding of model fitness for purpose.
