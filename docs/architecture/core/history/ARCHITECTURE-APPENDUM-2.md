# Architecture

**Status:** Historical
**Current reference:** ../ARCHITECTURE.md


> **Note:** This document is a revised architecture draft generated from the existing
architecture and the design decisions made during the v3.2.4 checkpoint experiments.
It is intended to replace the current ARCHITECTURE.md after review.

# Architecture

This document describes the conceptual architecture of Lumen.

It defines the major architectural components, their responsibilities and the relationships
between them. The architecture is intentionally independent of any programming language,
language model or deployment environment.

Capabilities are identified as:

- **Current** — implemented.
- **In Development** — actively being developed.
- **Architectural Direction** — planned.

---

# Principles for Prompt Design During Long-Running Tasks

The quality of a model's observable reasoning depends not only on the wording of individual prompts but also on how those prompts relate to the model's current cognitive state.

The following principles have emerged through experimentation during the development of Lumen.

---

## Minimise ambiguity

Prompts should communicate a clear objective while allowing the model sufficient freedom to determine the most appropriate response.

Overly prescriptive prompts can unnecessarily constrain reasoning, while ambiguous prompts may produce inconsistent behaviour.

The objective should be explicit, but the path to achieving it should remain flexible unless a specific methodology is required.

---

## Avoid unintended contradictions

Every new instruction should be considered in the context of all preceding System and Assistant prompts.

Even when individual instructions appear reasonable in isolation, their combined effect may alter the model's overall reasoning behaviour.

Prompt changes should therefore be evaluated as changes to an interacting system rather than as isolated edits.

---

## Respect the model's current cognitive state

Prompts should ask only questions that can be answered from the information currently available to the model.

A prompt should not require knowledge that has not yet been observed.

Incremental reasoning should therefore be grounded in present understanding rather than future prediction.

---

## Avoid asking the model to predict future understanding

An incremental checkpoint should preserve the model's current understanding.

It should not require the model to speculate about conclusions that may emerge only after additional source material has been analysed.

Questions concerning future architectural understanding, unresolved questions, or likely refactoring opportunities may be appropriate after task completion, but are generally unsuitable during incremental analysis.

---

## Recognise the difference between human and model information acquisition

Human engineers rarely analyse software by reading every source file sequentially.

Instead they navigate selectively, moving between functions, types, call sites, documentation and related files until they develop sufficient understanding.

By contrast, an LLM performing incremental source analysis acquires knowledge in the order information is presented.

Its understanding therefore develops cumulatively rather than through selective exploration.

Prompt design should acknowledge this difference rather than assuming human reading strategies.

---

## Ask questions appropriate to the current stage of analysis

The usefulness of a question depends upon when it is asked.

For example:

- "What does the model currently understand?" is appropriate throughout incremental analysis.

- "What architectural refactoring is required?" may only become meaningful after the relevant architecture has been observed.

Prompt timing is therefore as important as prompt wording.

---

## Evaluate behavioural impact rather than prompt wording

Small prompt changes can produce disproportionately large changes in observable reasoning behaviour.

Prompt optimisation should therefore be evaluated experimentally rather than intuitively.

The objective is not simply to produce better wording, but to produce better observable reasoning.

---

## Respect how the model acquired its knowledge

Prompt design should respect not only what the model knows, but how it came to know it.

Large Language Models performing incremental analysis acquire knowledge sequentially as information is presented.

Unlike a human engineer, they cannot arbitrarily navigate to unseen portions of the source material to answer a question.

Consequently, prompt design should not assume the model possesses information that has not yet been encountered.

Questions requiring knowledge beyond the model's current observable state should generally be deferred until sufficient source material has been analysed.

The sequence in which knowledge is acquired is therefore an important consideration when designing prompts for long-running tasks and Cognitive Checkpoints.

---

# Summary

These principles have emerged through observation and experimentation rather than theoretical prompt design.

They should therefore be regarded as engineering guidance rather than immutable rules and should continue to evolve as further behavioural experiments are performed.

The overarching objective remains unchanged:

> **Prompt design should maximise the quality of a model's observable reasoning while remaining consistent with the model's current state of knowledge.**

Lumen therefore treats prompt engineering as an evidence-based engineering discipline, where prompt behaviour is measured, compared, and refined through repeatable experimentation rather than intuition.

---

## Lumen Console

The Lumen Console is the primary operational interface for a running Lumen instance.

Its architecture, philosophy and operational model are documented separately in:

docs/LUMEN-CONSOLE.md