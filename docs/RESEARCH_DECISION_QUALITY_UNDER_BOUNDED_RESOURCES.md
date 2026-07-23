# Decision Quality Under Bounded Resources (DQBR)

This document proposes a research direction rather than a completed methodology. No accepted definition, scale, benchmark, or implementation currently exists. The purpose of this document is to preserve the concept, define its motivation, and establish a foundation for future investigation.

---

Status:
Research

---

## Abstract

Current AI benchmarks primarily measure model capability. They generally do not
measure whether a model selected an appropriate course of action given the
resources available to it.

This document proposes Decision Quality Under Bounded Resources (DQBR) as a
research direction for evaluating AI systems based upon the quality of their
decisions rather than raw capability.

DQBR is currently a conceptual framework and should not be considered a formal
metric.

---

## Motivation

Modern AI systems operate under constraints.

These include:

• context window
• latency
• available memory
• compute
• energy
• API cost
• hardware
• available tools
• available information
• user objectives

Current benchmarks typically ignore these constraints.

Real-world systems cannot.

---

## Inspiration

The work draws inspiration from Herbert Simon's work on

- Bounded Rationality
- Satisficing

Rather than assuming perfect optimisation, Simon argued that intelligent agents
make satisfactory decisions within finite resources.

This document explores whether similar ideas can be applied to AI evaluation.

---

## Current AI Evaluation

Explain

MMLU
HumanEval
SWE Bench

etc.

Explain what they measure.

Explain what they do not measure.

---

## Proposed Concept

Decision Quality

↓

under

↓

Bounded Resources

Rather than asking

"Can the model solve the task?"

DQBR asks

"Did the model choose an appropriate course of action given the resources
available?"

---

## Resources

List every bounded resource.

Context

Time

Latency

Tokens

Hardware

Cost

Available tools

Memory

Knowledge available

Human interaction

etc.

---

## Decision Variables

Possible variables include

- model selection
- tool selection
- retrieval depth
- stopping point
- reasoning depth
- context usage
- information gathering
- confidence
- uncertainty
- verification

---

## Example

Compare

Qwen 14B

Claude Opus

GPT

under identical tasks but different hardware and latency budgets.

The "best" model may differ despite identical tasks.

---

## Relationship to Satisficing

Explain that satisficing is not "good enough."

It is an intelligent stopping rule.

This becomes particularly relevant to AI systems which must determine

- when to retrieve more
- when to call tools
- when to stop reasoning
- when additional computation is unlikely to improve outcomes

---

## Potential Research Questions

Can DQBR become measurable?

Can decision quality be separated from model capability?

Can orchestration improve DQBR independently of model size?

Can smaller models outperform larger models under constrained resources?

Can DQBR assist automatic model selection?

Can DQBR assist tool selection?

Can DQBR assist orchestration systems?

---

## Challenges

No accepted scale exists.

Decision quality is multi-dimensional.

Ground truth is difficult.

Human satisfaction is subjective.

Resource costs vary.

The framework may require entirely new benchmark methodologies.

---

## Future Work

This document captures an early research direction.

The framework should evolve through experimentation, discussion and empirical
evaluation.

No formal implementation currently exists.