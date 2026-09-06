# System Prompts as Interpretive Frames

## Research Note --- Lumen

**Status:** Research / conceptual exploration\
**Area:** Moderari, Aestimare, provenance, behavioural assessment\
**Date:** 2026-08-31

## Premise

A system prompt can alter how a model interprets and responds to a task
without necessarily changing the model's underlying knowledge or
capability.

A useful human analogy is two people with different professional roles
discussing the same product: a salesperson and an engineer.

If both are asked:

> What is the most important thing about this product?

the salesperson may answer in terms of customer value, differentiation,
adoption, commercial benefit, and business outcomes.

The engineer may answer in terms of architecture, reliability,
interfaces, performance, maintainability, and technical constraints.

The answers may be substantially different while both remain correct.
The difference arises because the same subject is being interpreted
through different professional frames.

A system prompt can produce a similar effect in a language model.

## System Prompt as an Interpretive Frame

A system prompt can establish or influence:

-   the role the model is expected to perform;
-   which aspects of a problem deserve attention;
-   what constitutes a satisfactory answer;
-   which information should be prioritised;
-   what risks or constraints should receive greater weight;
-   which tools or capabilities should be considered;
-   how cautious, analytical, concise, critical, creative, or
    explanatory the response should be.

This suggests a useful distinction between three related concepts.

### Knowledge

What information and capability the model possesses or can access.

### Understanding and Context

How the model currently represents the task, conversation, evidence, and
surrounding situation.

### Interpretive Frame

What the model has been instructed or conditioned to regard as important
when converting that understanding into reasoning and an answer.

The system prompt is therefore not simply a formatting or behavioural
configuration. It can contribute directly to the interpretive frame
through which the model approaches a task.

## Example

Consider the same model receiving the same user prompt:

> Assess this proposed database migration.

Under one system prompt:

> You are a senior software engineer. Prioritise technical correctness,
> failure modes, reliability, and operational risk.

The response may concentrate on migration strategy, rollback mechanisms,
consistency, downtime, validation, performance, and failure recovery.

Under another system prompt:

> You are a product executive. Prioritise customer impact, delivery
> time, business continuity, and commercial consequences.

The response may concentrate on customer disruption, launch risk,
opportunity cost, schedule, revenue exposure, and stakeholder
communication.

Both responses may be valid.

The difference does not necessarily demonstrate inconsistency, error, or
a change in the model's underlying capability. It may demonstrate the
expected behavioural consequence of a changed interpretive frame.

## Causal Chain

A useful conceptual model is:

**System Prompt → Interpretive Frame → Model Understanding / Attention →
Reasoning Behaviour → Answer**

The system prompt therefore forms part of the **causal provenance of the
answer**.

An assessment of model behaviour that ignores the system prompt risks
attributing differences to the model that were actually introduced by
orchestration.

## Implications for Lumen

This distinction is particularly relevant to **Moderari** and
**Aestimare**.

Moderari controls or participates in the system-prompt environment
presented to the model. Consequently, Lumen should preserve the
effective system prompt associated with an execution as provenance
rather than treating it merely as transient configuration.

Aestimare should not assume that response divergence necessarily
represents behavioural instability.

Two executions may differ because of:

1.  stochastic variation;
2.  different model or model version;
3.  different context;
4.  different available evidence;
5.  different tools or tool results;
6.  different inference parameters;
7.  different system prompts;
8.  different interpretive frames intentionally created by
    orchestration.

This introduces an important assessment question:

> **Did the answers differ because the model behaved inconsistently, or
> because the same underlying capability was expressed through a
> different interpretive frame?**

That distinction may be essential when characterising model behaviour.

## Proposed Controlled Experiment

A useful Aestimare experiment would deliberately vary only the system
prompt while holding other relevant variables constant.

### Controlled Variables

Keep constant:

-   model and model version;
-   user prompt;
-   conversation context;
-   inference parameters;
-   available tools;
-   tool results;
-   external evidence;
-   execution environment, where practical.

### Experimental Variable

Change only the system prompt or explicitly defined role.

Example frames:

-   neutral assistant;
-   software engineer;
-   salesperson;
-   product executive;
-   security specialist;
-   risk assessor.

### Assessment Questions

Aestimare could then examine:

-   Which facts remain stable across frames?
-   Which facts or considerations receive greater or lesser emphasis?
-   Does the model reach different conclusions?
-   Are the different conclusions mutually compatible?
-   Does framing introduce unsupported claims?
-   Does framing suppress relevant evidence?
-   Does the model preserve factual correctness while changing emphasis?
-   Which behavioural characteristics are invariant across frames?
-   Which characteristics are system-prompt dependent?

This begins to distinguish **model capability** from **contextually
expressed model behaviour**.

## Research Significance

System prompts are often treated as instructions surrounding an LLM
invocation. For reasoning assurance, that treatment may be insufficient.

If a system prompt changes the interpretive frame through which the
model evaluates evidence and constructs an answer, it becomes part of
the explanation for why that answer exists.

Therefore:

> **The system prompt is part of the provenance of reasoning, not merely
> the configuration of execution.**

For Lumen, preserving and comparing system prompts may consequently be
necessary for explaining behavioural differences across traces, replays,
models, and longitudinal assessments.

This also suggests that system-prompt sensitivity itself could become a
measurable behavioural characteristic: not merely whether an answer
changes when framing changes, but **how, where, and to what degree the
model's reasoning behaviour changes in response to different
interpretive frames**.
