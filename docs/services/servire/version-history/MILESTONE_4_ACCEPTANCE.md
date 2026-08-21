# Milestone 4 Acceptance — Configuration Inspection & Preflight

## Objective

Make Servire capable of explaining the exact operational configuration that will be used before a managed process is launched.

## Delivered

- Read-only configuration inspection for every configured service.
- Expandable service configuration cards.
- Resolved working directory, Python executable and launch command display.
- Component config-file path and read-only file-content viewer.
- Health endpoint and dependency display.
- Resolved Servire YAML view.
- Runtime override placeholder for later client/model selection work.
- Collapsible validation details with persistent summary and timestamp.
- Validation report retained for the lifetime of the Servire process.
- Extended preflight checks for:
  - service roles;
  - dependency resolution;
  - working-directory existence;
  - working-directory writability;
  - Python executable existence;
  - component configuration readability;
  - launch target/module configuration;
  - health endpoint syntax.

## Deliberately Deferred

- Network availability probing of Pi, Ollama and MongoDB.
- Stack-level Start / Stop / Restart.
- Editing configuration.
- Runtime client/model selection.
- Unified log filtering.

## Acceptance Criteria

- Configuration inspection is read-only.
- Validation details can be collapsed and reopened without rerunning validation.
- Last validation summary remains visible after collapse and across subsequent dashboard requests.
- Component configuration files can be inspected from Servire when configured and readable.
- Resolved launch details match the configured working directory and virtual environment.
- Quality gates remain at or above the established project standards.
