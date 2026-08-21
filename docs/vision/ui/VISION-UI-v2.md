# VISION-UI-v2

**Status:** Historical
**Current reference:** ../../architecture/core/ARCHITECTURE.md


# Engineering Observability for AI Systems

## Vision

Lumen aims to make AI systems understandable while they are executing. The user interface is therefore designed around operational awareness rather than visual complexity.

## Why Observability Matters

Traditional applications expose logs after execution. AI orchestration benefits from continuous visibility into execution, reasoning progress and system state.

## Engineering Before Interface

The interface should reflect the architecture rather than define it. Every item displayed should correspond to a meaningful concept within the execution engine.

## Live vs Historical Information

The UI distinguishes between:

- Live operational state
- Historical artefacts

Operational state answers *what is happening now*. Historical artefacts explain *what happened previously*.

## Progressive Disclosure

The default view should present only the most important operational information. More detailed diagnostics should be available without overwhelming the operator.

## Operational Desktop

The primary dashboard should provide a single-screen view of:

- Current objective
- Current capability
- Active model
- Context utilisation
- Source analysis progress
- Replay and recovery state
- Checkpoint lifecycle
- Persistence status

The design philosophy is comparable to tools such as **btop**: immediate understanding through concise presentation.

## Human-Centred Engineering

The UI exists to reduce cognitive load. Engineers should spend time understanding systems rather than searching for information across multiple log files.

## Beyond Logs

Logs remain valuable for forensic analysis, but they should not be the primary operational interface. The dashboard should expose the information engineers routinely need without requiring log inspection.

## Multi-Model Future

As Lumen evolves into a multi-model orchestration platform, the UI should present providers, models and capabilities consistently, regardless of implementation technology.

## Long-Term Direction

The long-term goal is an engineering workstation for AI orchestration where continuity, execution and observability are treated as first-class architectural capabilities.

Rather than monitoring isolated components, the interface should present the operational state of the entire orchestration engine as a coherent system.
