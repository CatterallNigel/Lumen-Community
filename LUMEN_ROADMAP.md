## Roadmap Process

Ideas begin in **WANT**.

Once an idea has sufficient technical justification and a defined implementation approach, it moves to **DO**.

When a capability becomes fundamental to Lumen's architecture or is required before broader release, it moves to **NEED**.

Completed implementation should result in the creation or update of the appropriate architectural or design documentation.

The roadmap describes planned engineering work. Architecture, research and future vision are documented separately and referenced where appropriate.

---

## Product Philosophy 

Models perform reasoning — Lumen preserves project continuity.

---

# NEED

These capabilities are considered fundamental to Lumen's architecture and are required before broader release.

## Session Persistence
Persist every session so work survives failures, application restarts and extended periods of inactivity.

## Continuity Checkpoints
Capture immutable continuity checkpoints throughout a session to preserve project continuity across context boundaries.

## Session Resume
Resume previous work by reconstructing project continuity rather than replaying conversation history.

## Session Lifecycle
Treat sessions as first-class objects with creation, continuation, branching and completion.

## Context Management
Continuously monitor available context and perform controlled continuity preservation before context exhaustion.

## Objective Continuity
Maintain and continually reinforce project objectives, established facts, constraints and architectural decisions throughout long-running work.

## Recovery Journal
Persist sufficient operational state to recover safely after unexpected interruption or failure.

## Source Revalidation
Treat original source material as authoritative when resuming work or validating continuity.

## Protected Continuity
Protect inherited continuity from degradation during later checkpoint creation and context compaction.

## Long Running Task Support
Support engineering and knowledge-work tasks that span many hours, multiple sessions or multiple days.

## Operational Observability

Expose the internal state of Lumen through transparent inspection of sessions, continuity, checkpoints, context utilisation, provenance, model interactions and operational metrics, ensuring that long-running AI workflows remain understandable, auditable and diagnosable.

---

# DO

These capabilities are planned engineering work that builds upon the core architecture defined in **NEED**.

## Automatic Client Startup
Automatically initialise supported clients and establish a ready-to-use Lumen environment.

## Single Command Startup
Provide a single command that starts the complete Lumen environment, including models, services and client integration.

## Session Commands
Provide commands for creating, listing, inspecting, recalling and managing projects and sessions.

## Immutable Checkpoint History
Maintain a permanent, chronological history of continuity checkpoints for every project.

## Checkpoint Timeline
Provide visual navigation through checkpoint history, allowing project continuity to be inspected and compared over time.

## Working State Inspection
Provide detailed inspection of the model's current operational state, including active objectives, continuity, context utilisation and checkpoint information.

## Distilled Cognition Viewer
Provide a live view of the model's distilled continuity state and checkpoint evolution throughout long-running tasks.

## Efficiency Report
Report operational metrics including token usage, execution time, context utilisation and continuity efficiency to improve observability and prompt optimisation.

## Continuity Quality Metrics
Measure and report the quality and completeness of preserved project continuity across sessions and checkpoints.

## Multiple Provider Support
Support multiple model providers through a common provider abstraction while maintaining consistent Lumen behaviour.

---

# WANT

These capabilities represent the long-term direction of Lumen once the core platform has matured. They are architectural goals rather than immediate engineering priorities.

## Automatic Session Rollover
Seamlessly transition between model sessions as context limits are reached while preserving complete project continuity.

## Multi-Agent Orchestration
Coordinate multiple specialised agents working towards shared project objectives while maintaining common continuity and provenance.

## Verification Agents
Introduce independent verification agents capable of reviewing continuity, validating reasoning and identifying inconsistencies.

## Prompt Optimisation Framework
Continuously evaluate prompt effectiveness using controlled experimentation and objective quality metrics.

## Provider Intelligence
Automatically select the most appropriate model provider based on task characteristics, capability, performance and cost.

## Distributed Execution
Allow Lumen to coordinate work across multiple machines and multiple model providers.

## Multi-User Platform
Support multiple concurrent users, projects and organisations with complete isolation of continuity, permissions and project state.

## Virtual Unlimited Context
Provide seamless project continuity across an unlimited number of underlying model sessions.

## Cognitive State Evolution
Analyse how model understanding evolves throughout long-running projects and identify significant changes in reasoning behaviour.

## Project Knowledge Graph
Build an interconnected representation of project objectives, decisions, files, sources, continuity and provenance to improve navigation and reasoning.

## Plugin and Tool Ecosystem
Provide a stable extension framework allowing external tools, providers and integrations to become first-class Lumen components.

