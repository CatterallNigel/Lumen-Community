# Lumen Architecture Proposal
## Servire as the Unified User Console

**Status:** Architectural Discussion  
**Version:** Draft 1.0  
**Date:** August 2026

---

# Purpose

This document captures a proposed future architectural direction for the Lumen++ ecosystem.

It is **not** an implementation plan and is **not** currently scheduled for development. It records an architectural idea for future evaluation once Replay, Trace and Assess have matured.

---

# Background

Today the user primarily interacts with the ecosystem through Pi.

Simplified architecture:

```text
User
  │
Pi
  │
Lumen
  │
Model
```

Replay, Trace and Servire exist alongside this execution path.

As the ecosystem has evolved, Servire has become the operational control plane while Pi continues to provide both the user interface and tool execution.

These are arguably two different responsibilities.

---

# Proposed Direction

Move ownership of the primary user interface from **Pi** to **Servire**.

Servire becomes the single entry point for both operators and users.

Pi evolves into a backend service responsible solely for tool execution.

The execution path becomes:

```text
User
  │
Servire
  │
Lumen
  │
Pi (Tool Provider)
  │
Model
```

Lumen remains the sole orchestration engine.

---

# Architectural Responsibilities

| Component | Responsibility |
|-----------|----------------|
| **Servire** | User interface and operational workbench |
| **Lumen** | AI orchestration and continuity |
| **Pi** | Tool provider |
| **Replay** | Behaviour reproduction |
| **Trace** | Behaviour recording |
| **Assess** | Behaviour evaluation |

This preserves the single-responsibility principle across the ecosystem.

---

# Why Move the Console?

A unified console provides a single place to interact with every aspect of Lumen++.

Potential workspaces include:

- Conversation
- Operations
- Replay
- Trace
- Assess
- Checkpoints
- Configuration
- Logs

Rather than switching between independent applications, the user works entirely within Servire.

---

# Pi as a Tool Provider

Under this proposal Pi no longer owns the user experience.

Instead Pi becomes infrastructure providing services such as:

- Tool discovery
- Tool execution
- Tool metadata
- Tool status
- Tool capability reporting

Conceptually Pi becomes similar to MongoDB or Ollama: an external service consumed by Lumen.

---

# Future Benefits

Decoupling the console from the tool provider introduces significant architectural flexibility.

Examples include:

## Alternative Tool Providers

Pi could be replaced without changing the Servire user experience.

Possible future providers might include:

- Pi
- OpenAI tool execution
- MCP-compatible providers
- Custom enterprise tool services

## Multiple Tool Providers

Rather than selecting one provider, Lumen could support several simultaneously.

For example:

```text
Lumen
  │
 ├── Pi
 ├── MCP Server A
 ├── MCP Server B
 ├── Internal Enterprise Tools
 └── Future Tool Provider
```

Lumen would select the appropriate provider according to capability, configuration or policy.

Servire remains unaware of which provider ultimately executes a tool.

---

# Architectural Advantages

Separating the console from tool execution provides:

- clearer component boundaries
- interchangeable tool providers
- support for multiple providers
- easier future integration
- reduced coupling
- cleaner testing
- improved long-term maintainability

Most importantly, it prevents the user interface from becoming tightly coupled to a single tool ecosystem.

---

# Important Architectural Constraint

This proposal does **not** change the orchestration model.

Servire must never become an orchestration engine.

Correct execution remains:

```text
Servire
    ↓
Lumen
    ↓
Tool Provider(s)
    ↓
Model
```

Lumen continues to own:

- orchestration
- continuity
- routing
- decision making
- context management

Servire owns only the user experience and operational management.

---

# Long-Term Vision

If adopted, Servire evolves from an operational control plane into the **Lumen++ Workbench**.

It becomes the single desktop environment for the ecosystem while remaining architecturally separate from execution.

This also establishes a stable abstraction around tool providers, allowing Pi to evolve independently or be replaced entirely without affecting the user experience.

In the longer term, supporting multiple tool providers simultaneously may prove more valuable than supporting only one. This proposal intentionally keeps that option open while preserving the existing responsibilities of every major Lumen++ component.
