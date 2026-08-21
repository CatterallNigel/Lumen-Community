# Lumen Vision: Out-of-the-Box Experience

## Purpose

One of Lumen's primary design goals is to minimise the effort required to begin using it.

Users should not be expected to understand the relationships between clients, model providers, APIs, models, or configuration before they can be productive.

Instead, installing Lumen should guide the user through discovering their existing environment, identifying any missing components, configuring everything automatically where possible, and leaving the user with a fully functioning AI development environment.

The objective is simple:

> **Install Lumen. Answer a few questions. Start working.**

---

# Design Principles

The installation experience should be:

* Simple
* Transparent
* Automated where possible
* Non-destructive
* Vendor neutral
* Fully configurable for advanced users

Lumen should explain what it is doing, never silently modify a user's environment, and always allow experienced users to customise the configuration.

---

# Installation Philosophy

Lumen is the product.

Ollama, LM Studio, AI models, and clients such as Pi are supporting components.

The installation process should therefore revolve around configuring **Lumen**, rather than requiring users to manually assemble multiple independent applications.

---

# Environment Discovery

After installation, Lumen performs a discovery phase.

Typical checks include:

* Operating system
* Available memory
* CPU and GPU capabilities
* Existing Lumen configuration
* Installed model providers
* Installed AI clients
* Existing local models
* Network availability
* Local storage locations

The discovery phase should never make changes.

Its purpose is simply to understand the current environment.

---

# Model Provider Detection

Lumen searches for supported model providers.

Initially these include:

* Ollama
* LM Studio

For each provider Lumen determines:

* Is it installed?
* Is it currently running?
* Can its API be reached?

For example, if Ollama is installed but not running, Lumen may offer to start it automatically.

If no supported provider is found, Lumen should present suitable options, including download links or installation guidance.

---

# Model Discovery

Once a provider is available, Lumen queries it for installed models.

For example:

```
Available models

• qwen2.5-coder:14b
• gemma3:4b
• devstral:24b
```

If no models are installed, Lumen should recommend an appropriate starting model based on the machine's available resources and the user's intended workload.

Recommendations should always include an explanation of why a particular model is being suggested.

---

# Client Discovery

Lumen next searches for supported AI clients.

Initially this includes:

* Pi

If Pi is available, Lumen offers to configure it automatically.

If Pi is not installed, Lumen asks whether the user wishes to:

* install Pi
* configure another OpenAI-compatible client
* use Lumen as an API service only

Future versions may support automatic configuration of additional clients.

---

# Automatic Client Configuration

When Pi is selected, Lumen configures it to communicate directly with Lumen.

The resulting architecture becomes:

```
Pi
 │
 ▼
Lumen
 │
 ▼
Ollama / LM Studio
 │
 ▼
Selected Model
```

From the user's perspective, Pi simply communicates with an OpenAI-compatible endpoint.

Lumen performs all orchestration behind the scenes, including:

* provider abstraction
* model selection
* context monitoring
* checkpoint creation
* session continuity
* project continuity
* tool orchestration
* future memory retrieval
* future auditing

The client does not need to understand any of these features.

---

# Guided Setup

The complete installation experience should resemble a guided assistant.

Example flow:

```
Install Lumen

↓

Discover Environment

↓

Detect Model Providers

↓

Detect Installed Models

↓

Recommend or Download a Model

↓

Detect AI Client

↓

Configure Client

↓

Verify Configuration

↓

Ready
```

The installer should present clear progress and explain each step before making any changes.

---

# Verification

Before completion, Lumen verifies that every component can communicate successfully.

Typical checks include:

* Lumen service running
* Provider responding
* Selected model available
* Client correctly configured
* Test prompt successfully executed

Example summary:

```
Lumen Setup Complete

Client:
    Pi

Provider:
    Ollama

Model:
    qwen2.5-coder:14b

Status:
    Ready
```

The user should be able to begin working immediately.

---

# Advanced Users

Although the default installation should be highly automated, advanced users must retain full control.

They should be able to:

* use alternative clients
* use remote model providers
* configure OpenAI-compatible endpoints
* select custom models
* alter storage locations
* disable automatic configuration
* manually edit configuration files

Automation should simplify installation, never reduce flexibility.

---

# Long-Term Vision

Ultimately, installing Lumen should feel comparable to installing a modern development tool.

Rather than reading multiple setup guides and manually connecting components, users should install Lumen, answer a small number of questions, and immediately begin working with a fully configured AI environment.

This experience reinforces Lumen's role as the orchestration layer that connects clients, models, providers, memory, continuity, and tooling into a cohesive and model-agnostic platform.

The guiding principle is straightforward:

> **Install Lumen. Configure once. Work immediately.**
