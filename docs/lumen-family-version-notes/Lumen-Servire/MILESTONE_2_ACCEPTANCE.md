# Milestone 2 Acceptance — Service Catalogue

## Objective

Establish the service metadata and configuration-validation model that later Servire milestones will use for process control and stack orchestration.

## Delivered

- Explicit service roles:
  - Client
  - Model Provider
  - Persistence
  - Orchestrator
  - Replay Engine
  - Recorder
- Managed and external service modes.
- Configurable working directories and virtual-environment Python executables.
- Script and Python-module launch definitions.
- Launch arguments, health endpoints, launch hints and role dependencies.
- Service catalogue capable of resolving implementations by role.
- Role-based dependency graph.
- Configuration-only stack validation.
- Long-term dashboard layout:
  - Stack Health
  - Stack Actions
  - Managed Services / External Dependencies
  - Operational Log
  - Configuration
- Stack Actions toolbar containing:
  - Validate
  - Start
  - Stop
  - Restart
- Validate is active in Milestone 2; process actions remain disabled until later milestones.
- Validation results are presented directly on the dashboard.

## Validation Scope

Milestone 2 validation checks:

- all required architectural roles have configured implementations;
- configured role dependencies resolve;
- managed-service working directories exist;
- configured virtual-environment Python executables exist;
- script launch targets exist;
- module launch targets are structurally configured.

External runtime availability is deliberately not probed in Milestone 2. Pi, Ollama and MongoDB are represented as external prerequisites, with active availability checks deferred to a later milestone.

## Architectural Decisions

Servire reasons about service **roles**, not product names. Pi, Ollama and MongoDB are the current implementations of Client, Model Provider and Persistence; they are not permanent architectural dependencies.

The current managed Lumen++ stack remains:

1. Lumen — Orchestrator
2. Replay — Replay Engine
3. Trace — Recorder

The current external prerequisites are:

1. Pi — Client
2. Ollama — Model Provider
3. MongoDB — Persistence

No process lifecycle control is introduced by Milestone 2.

## Quality Gate Target

- pytest: pass
- coverage: >= 95%
- Ruff: clean
- mypy: clean

The implementation package was exercised with 23 passing tests and 99.42% coverage in the build environment. Ruff and mypy should be rerun in the project's Python 3.12.10 virtual environment as the authoritative local quality gates.
