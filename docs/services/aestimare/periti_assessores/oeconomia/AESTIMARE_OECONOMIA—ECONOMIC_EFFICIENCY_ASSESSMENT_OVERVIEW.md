# Aestimare Oeconomia
## Economic Efficiency Assessment Overview

### Status

Exploratory architecture and research concept.

This document describes a prospective specialist assessor within **Lumen Aestimare** concerned with the economics and efficiency of model use.

The working name is:

**Aestimare Oeconomia — Economic Efficiency Assessor**

Oeconomia would operate as one of Aestimare's specialist **Periti Assessores**, independently examining economic and interaction-efficiency evidence while Aestimare coordinates its findings with those of other assessors.

---

## 1. Premise

The basic economics of using an AI model are commonly expressed in terms of token price:

- cost of input tokens;
- cost of output tokens.

These are important and readily measurable, but they do not represent the complete economic cost of obtaining a useful result.

A low-cost model may require a carefully constructed and highly descriptive prompt, several clarification turns, retries, or substantial human interpretation of its output.

A more expensive model may understand a comparatively simple request immediately and provide a concise, sufficient response.

Conversely, using a highly capable and expensive frontier model for a task that a much smaller model can perform equally well represents unnecessary expenditure.

The economic question is therefore not simply:

> **How much does this model cost?**

Nor even:

> **How much did this execution cost?**

The more useful question is:

> **What was the total cost of obtaining a satisfactory outcome for this particular task?**

---

## 2. Cost per Successful Task

Cost per successful task is a considerably more useful measure than raw token pricing.

However, even this can understate the real cost of model use if it considers only computational expenditure.

A successful interaction imposes costs in several places.

### 2.1 Machine Cost

Directly measurable computational costs include:

- input tokens;
- output tokens;
- model/API pricing;
- inference resources;
- tool calls;
- retries;
- repeated executions;
- orchestration overhead;
- execution time.

These provide the most obvious economic measurements.

They do not provide the whole picture.

### 2.2 Human Input Cost

The user must communicate the task to the model.

Different models may require substantially different amounts of context, precision and explanation before they can successfully perform the same task.

Human input cost can therefore include:

- prompt length;
- contextual information required;
- prompt refinement;
- clarification turns;
- repeated instructions;
- corrections;
- specialist prompt construction;
- time spent formulating the request.

A computationally inexpensive model may therefore be expensive to operate if the user must invest significant effort in making it understand the task.

### 2.3 Human Consumption Cost

The model's output also imposes a cost.

Models frequently produce responses containing more information than is necessary to fulfil the user's objective.

The direct consequence is additional output-token expenditure.

Potentially more important is the human cost.

The user must:

- read the response;
- understand it;
- identify the relevant information;
- distinguish useful information from padding;
- potentially reformulate or summarise it;
- decide whether the original question was actually answered.

A verbose but correct answer can therefore be economically inferior to a concise answer of equivalent useful quality.

---

## 3. Brevity Is Not the Objective

Oeconomia should not simply reward shorter responses.

That would create an undesirable optimisation target.

A response can be extremely concise and completely inadequate.

Likewise, a long response may be entirely justified by the complexity of the task.

The desirable property is therefore not brevity itself, but something closer to:

**sufficient brevity**

or:

**information efficiency subject to task sufficiency.**

The relevant question becomes:

> **How much communication was required to satisfactorily fulfil the task?**

A twenty-token response to a task requiring careful explanation may be poor.

A two-thousand-token response to a task satisfactorily answerable in two hundred tokens may also be poor.

The appropriate response is the one that communicates sufficient information without imposing unnecessary computational or human consumption cost.

---

## 4. Prompt Efficiency

A similar principle applies to input.

More capable models may require less carefully engineered prompts to understand a user's intent.

This creates a potentially measurable property:

**prompt efficiency**.

For example, the same task could be expressed using progressively richer prompts:

- minimal ordinary-language request;
- request with additional context;
- carefully specified request;
- highly engineered and explicit prompt.

Different models could then be assessed according to the point at which they begin reliably satisfying the task.

This creates something resembling a **context requirement curve**.

A relatively inexpensive model might require extensive contextual specification before succeeding reliably.

A more expensive model might satisfactorily infer the same requirement from a short ordinary-language request.

The more expensive model may therefore have the lower total economic cost once human interaction is considered.

---

## 5. Three Forms of Efficiency

This leads to an important distinction for Aestimare.

### Computational Efficiency

How economically did the machine execute the task?

This includes tokens, inference resources, execution time and external service costs.

### Interaction Efficiency

How efficiently did the human and model reach the required result?

This includes prompting, clarification, retries, response length and comprehension effort.

### Economic Efficiency

Was the total expenditure of machine and human resources justified by the quality and usefulness of the result?

These should not be assumed to be equivalent.

A model can be computationally inexpensive while being expensive to use.

A model can also be computationally expensive while being economically efficient because it significantly reduces human effort or achieves reliable first-pass completion.

---

## 6. Quality Must Remain Part of the Equation

Cost cannot be assessed independently of outcome quality.

An inexpensive incorrect answer has little economic value.

Likewise, repeatedly executing a cheap model until an acceptable result is obtained may ultimately cost more than using a more capable model once.

A conceptual measure might therefore be expressed as:

> **Economic Value = Useful Task Outcome / Total Cost of Obtaining and Consuming the Outcome**

This is not proposed as a final mathematical formula.

It describes the relationship Aestimare Oeconomia is intended to investigate.

The important point is that the denominator represents more than API expenditure.

---


## 7. The Economic Iceberg

Token cost is attractive as an economic measure because it is visible, deterministic and readily attributable.

For a hosted model, an organisation can usually calculate a direct execution cost from input-token pricing, output-token pricing and the number of tokens consumed. This produces a figure that can be forecast, budgeted and compared.

That certainty is useful, but it represents only the visible part of the economics of AI.

A more complete model resembles an **economic iceberg**:

```text
                    TOKEN COST
                 ───────────────
~~~~~~~~~~~~~~~~~~~ waterline ~~~~~~~~~~~~~~~~~~~

                Prompting effort
                Context preparation
                Clarification
                Retries and failures
                Human verification
                Reading/comprehension
                Tool/orchestration cost
                Workflow integration
                Model escalation
                Human time
                       |
                       v
               SATISFACTORY OUTCOME
                       |
                       v
                PRODUCTIVITY GAIN
                       |
                       v
                 REALISED VALUE?
```

The portion above the waterline is comparatively easy to measure.

The costs and benefits below it become progressively less direct, less attributable and more dependent upon the environment in which the model is being used.

This creates an important distinction for Oeconomia:

> **The visible economics of AI are token costs. The actual economics extend from the resources required to obtain a satisfactory outcome through to whether the resulting benefit is actually realised.**

### 7.1 Cost, Efficiency, Productivity and Value

Oeconomia should not collapse **cost**, **efficiency**, **productivity** and **value** into a single concept.

They describe related but different stages.

**Cost** describes the resources consumed.

**Efficiency** describes the relationship between resources consumed and the satisfactory outcome obtained.

**Productivity gain** describes an improvement in the amount, speed or quality of useful work that becomes possible.

**Realised value** describes whether that improvement ultimately produces a meaningful benefit for the organisation or user.

For example, an AI system may reduce the time required for a task from two hours to one hour. That represents a measurable productivity improvement.

It does not automatically follow that the saved hour produces economic value.

The saved capacity might be used for additional productive work, improve delivery time, reduce operating cost or increase quality. Alternatively, it may never translate into a benefit that the organisation captures.

The chain is therefore better represented as:

> **Resources consumed → Satisfactory outcome → Productivity effect → Realised value**

Each stage presents a different assessment question.

### 7.2 The Boundary of Oeconomia

The further assessment moves from direct execution cost toward organisational realised value, the more difficult causal attribution becomes.

Oeconomia should therefore avoid claiming precision that the available evidence cannot support.

Its strongest evidence may initially concern execution and interaction economics:

- token expenditure;
- model/API cost;
- execution time;
- retries;
- tool calls;
- prompt/context burden;
- response volume;
- clarification;
- model escalation; and
- other observable interaction costs.

Productivity effects may sometimes be measurable.

Realised organisational value may require evidence outside Lumen and may depend upon business processes, organisational behaviour and economic assumptions that Oeconomia cannot directly observe.

This does not make realised value irrelevant. It means that Oeconomia must identify the boundary between what it can measure and what it can only estimate or infer.

---

## 8. Evidence Confidence and Economic Claims

Economic assessment becomes dangerous when uncertain assumptions are converted into apparently precise numbers.

Oeconomia should therefore characterise not only an economic observation, but also the **evidential basis** on which that observation rests.

A useful initial distinction is:

### Measured

Directly observed evidence.

Examples include:

- token counts;
- API price;
- number of executions;
- number of retries;
- number of tool calls;
- response length;
- execution latency.

### Derived

A value calculated from measured evidence using a defined method.

Examples include:

- direct monetary token cost;
- average execution cost;
- first-pass completion rate;
- cost per satisfactory execution;
- comparative token expenditure.

### Estimated

A value based on an explicit approximation or model.

Examples might include:

- expected reading time;
- estimated prompting time;
- approximate human interaction cost.

The assumptions used to create the estimate should remain visible.

### Inferred

A conclusion suggested by the available evidence but not directly measured.

Examples might include:

- likely reduction in user effort;
- likely productivity improvement;
- probable economic advantage for a particular workload.

Inference should not be presented as measurement.

This distinction is fundamental to Oeconomia.

A precise-looking monetary figure should not imply a level of certainty that the underlying evidence does not possess.

> **Economic assessment should preserve uncertainty rather than hide it behind precision.**

This is particularly important when comparing models. A model can be demonstrably cheaper in direct token expenditure while the broader claim that it is economically superior may depend upon estimated human effort, task quality or organisational context.


## 9. Potential Evidence

Oeconomia could potentially evaluate evidence including:

| Dimension | Potential Evidence |
|---|---|
| Input cost | Input tokens and monetary cost |
| Output cost | Output tokens and monetary cost |
| Prompt burden | Amount of user input required |
| Context requirement | Context required before reliable success |
| Interaction burden | Number of clarification turns |
| Retry cost | Failed or repeated executions |
| Output efficiency | Useful information relative to response volume |
| Task success | Whether the objective was satisfactorily fulfilled |
| Model utilisation | Whether model capability was disproportionate to task requirements |
| Execution time | Model and orchestration latency |
| Human time | Prompting and consumption effort where measurable |
| Tool cost | Additional calls and external resources |
| Comparative cost | Cost relative to alternative models performing the same task |
| Productivity effect | Observable change in time, throughput or quality where evidence exists |
| Realised value | Organisational benefit where externally supportable evidence exists |
| Evidence confidence | Whether an economic claim is measured, derived, estimated or inferred |

Not all of these measurements will necessarily be directly observable.

Aestimare should distinguish measured evidence from inferred or estimated values.

---

## 10. Fitness for Purpose

This assessor should not attempt to determine the universally "best" model.

There may be no such model.

Instead, it should contribute evidence toward a more useful question:

> **Which model provides sufficient quality for this class of task at an appropriate total cost?**

For a trivial arithmetic task, invoking an expensive frontier reasoning model may provide no useful additional value over a small model capable of producing the correct answer reliably.

For a complex reasoning task, however, the frontier model may require substantially less prompting, fewer retries and less human verification.

The more expensive model may then be the economically preferable choice.

The appropriate model is therefore dependent upon the task.

**Model cost does not equate directly to model value.**

---

## 11. From Benchmarking to Evidence

Conventional model benchmarks are useful but necessarily generalised.

They cannot tell an individual organisation whether a particular model represents good economic value for its particular workload.

Aestimare has the opportunity to approach the problem differently.

Rather than relying solely upon benchmark reputation, model size or token pricing, it can accumulate evidence from actual executions.

Over time it may become possible to observe patterns such as:

> Task class A + Model X  
> High first-pass success  
> Low prompt burden  
> Low output burden  
> Higher token price  
> Lower overall interaction cost

while:

> Task class B + Model Y  
> Equivalent quality  
> Lower token price  
> Equivalent interaction burden  
> Lower overall economic cost

The question consequently changes from:

> **Which model is best?**

to:

> **Which model is economically appropriate for this task?**

---

## 12. Relationship to Aestimare

Oeconomia should not determine model suitability alone.

Economic efficiency is only one dimension of model behaviour.

Other Periti Assessores may evaluate qualities such as behavioural consistency, task sufficiency, divergence, context sensitivity or other dimensions developed as Aestimare evolves.

Aestimare can then combine these independent assessments.

This preserves an important architectural principle:

> **Economic efficiency is evidence about model suitability; it is not itself the definition of model quality.**

A cheap answer is not necessarily a good answer.

An expensive answer is not necessarily a better answer.

The relationship between expenditure and satisfactory outcome is what matters.

---

## 13. Longer-Term Possibility

Accumulated Oeconomia evidence could eventually become useful outside assessment itself.

If repeated controlled observations demonstrate that a particular model reliably performs a particular class of task at substantially lower total economic cost, that evidence could inform future model selection.

Moderari could eventually use such evidence when deciding which available model is appropriate for a task.

This would replace model selection based primarily on reputation, size or headline benchmark performance with selection based upon observed fitness for purpose.

Such behaviour is beyond the initial scope of Oeconomia.

The assessor's first responsibility is simply to **measure and characterise the evidence**.

---

## 14. Core Principle

The underlying proposition can be reduced to:

> **Cost per successful task still isn't enough if we don't account for the cost imposed on the human to obtain and consume that success.**

The economics of AI therefore includes not only the resources consumed by the model, but also the resources consumed by the interaction between the model and its user.

Token expenditure is the visible portion of that economic system. Prompting, context preparation, clarification, retries, verification, comprehension, integration and human time sit below the waterline, while productivity and realised value determine whether expenditure ultimately produced worthwhile benefit.

Oeconomia should measure what can be measured, derive what can be calculated, estimate only with explicit assumptions, and clearly identify what is inferred.

Aestimare Oeconomia exists to investigate that relationship without manufacturing certainty where the evidence does not support it.