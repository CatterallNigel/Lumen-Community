# UI_PRINCIPLES.md

# Lumen Replay UI Principles

Version: 1.0  
Status: Engineering Design Principle  
Applies to: Lumen Replay (Lumen Repetere)

---

# Purpose

The Replay user interface exists to make replay experiments transparent.

Replay is not simply an execution engine.

Replay is an operator tool whose purpose is to allow an engineer to understand exactly what was recorded, what will happen during replay, what is happening now, and what happened afterwards.

The UI should always favour visibility over abstraction.

---

# Core Principle

> **Never hide what the system is doing.**

Every significant operation performed by Replay should be visible to the operator.

If Replay performs an action that affects an experiment, the operator should be able to inspect it.

---

# The Five Questions

Every Replay screen should help answer one or more of these questions.

## 1. What was recorded?

Replay should provide complete visibility into the original Trace recording.

Examples include:

- recording metadata
- conversation timeline
- message sequence
- checkpoints
- headers
- request bodies
- response bodies
- timestamps
- durations
- statistics

The operator should never need to inspect MongoDB directly.

---

## 2. What is about to happen?

Before Replay performs an experiment the operator should understand:

- which recording will be replayed
- replay identifier
- replay session identifier
- experiment name
- target system
- configuration
- identifier mappings
- replay options

Preparation should always be visible before execution begins.

---

## 3. What is happening now?

During replay the UI should provide live operational visibility.

Examples include:

- current message
- current exchange
- progress
- elapsed time
- current replay state
- current checkpoint
- request being transmitted
- response received
- timing
- logging

Replay should feel like watching a debugger rather than waiting for a batch process.

---

## 4. What happened?

After execution Replay should preserve the experiment.

Examples include:

- completion status
- timings
- failures
- replay statistics
- captured responses
- execution logs

Nothing should disappear once execution completes.

---

## 5. Why?

Replay should always retain sufficient provenance for an operator to understand why an outcome occurred.

This includes links to:

- original Trace recording
- replay session
- configuration
- target
- timestamps
- execution logs

Replay should always explain its decisions.

---

# Visibility First

Visibility takes priority over convenience.

Replay should display information rather than requiring operators to retrieve it manually.

Examples:

Good

- Timeline visualisation
- Conversation explorer
- Message inspector
- Progress indicators
- Health indicators

Poor

- Hidden processing
- Silent failures
- Background actions without feedback
- "Magic" behaviour

---

# Recording Explorer

The Recording Explorer is the primary user experience.

It should become the equivalent of a debugger for recorded AI conversations.

The operator should be able to inspect:

- recording summary
- timeline
- conversation
- checkpoints
- metadata
- statistics
- message details
- binary payloads
- headers
- body content

The operator should never need to download JSON merely to understand a recording.

---

# Experiment Preparation

Preparation is a first-class operation.

Replay should expose:

Original Recording

↓

Replay Session

↓

Identifier Mapping

↓

Configuration

↓

Ready

The operator should understand exactly what Replay is preparing before execution begins.

---

# Execution

Execution should always be observable.

The UI should expose:

- replay progress
- current message
- current exchange
- current request
- current response
- timing
- latency
- replay state
- failures
- logs

The operator should never wonder what Replay is doing.

---

# Configuration

Configuration should always be visible.

Examples include:

- MongoDB connection
- target endpoint
- collection names
- replay configuration
- timeout values
- environment

Sensitive values such as passwords or tokens must always be masked.

---

# Health

Replay should clearly display the health of all external dependencies.

Examples include:

- MongoDB
- target system
- Replay service

Health should be visible before an operator attempts an experiment.

---

# Progressive Disclosure

The dashboard should provide summary information first.

Detailed inspection should always be available through drill-down views.

Dashboard

↓

Recording Explorer

↓

Conversation

↓

Message Inspector

The operator should never be overwhelmed but should always be able to reach complete detail.

---

# Immutable Provenance

Replay never modifies Trace recordings.

Trace records reality.

Replay adapts reality for experimentation.

Every Replay artefact must retain provenance back to the original recording.

---

# Separation of Responsibilities

Replay is responsible for:

- replay preparation
- replay execution
- replay visibility
- replay provenance

Replay is **not** responsible for:

- recording conversations
- behavioural assessment
- scoring
- orchestration

Those responsibilities belong respectively to:

- Trace (Vestigare)
- Assess (Aestimare)
- Servire

---

# Design Philosophy

Replay should feel less like an API and more like an engineering workbench.

Comparable tools include:

- Wireshark
- Fiddler
- Chrome Developer Tools
- Visual Studio Debugger

These tools succeed because they expose detail rather than hiding it.

Replay should do the same for AI conversations.

---

# Guiding Principle

> Replay should never perform a significant operation without first making it visible to the operator.

If an operator cannot answer:

- What was recorded?
- What is about to happen?
- What is happening now?
- What happened?
- Why?

then the UI should be improved until they can.