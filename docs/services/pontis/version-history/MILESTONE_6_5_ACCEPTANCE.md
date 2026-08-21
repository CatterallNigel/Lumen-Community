# Lumen Pontis M6.5 Acceptance — Originating Tool Ownership

## Purpose

Pontis now makes tool-result routing a property of the originating session rather than
inferring ownership when a model tool call returns.

## Invariant

When a session first enters Pontis:

- if the originating client declares a non-empty OpenAI-compatible `tools` collection,
  the originating client owns tool execution and Pontis preserves the originating HTTP
  response path;
- if the originating client does not declare tools, Pontis uses provider arbitration and
  may bind an ACP tool provider for that session.

For stable session identifiers this decision is sticky for the lifetime of the Pontis
session. Replay participation, a Replay fork, Trace recording, provider capability
enrichment, or a later request shape cannot change the established tool route.

## Behaviour validated

- Pi/tool-capable external client receives model tool calls directly over its originating
  HTTP route; Pontis does not create a second ACP Pi session.
- A later turn in the same stable tool-capable session remains on the originating route
  even when that later request omits a `tools` declaration.
- Rogare remains tool-less as an originating client. Pontis may enrich Rogare requests
  with provider model/tools for Moderari, but that enrichment does not transfer tool
  ownership to Rogare; model tool calls continue through the bound ACP provider.
- Replay session headers do not alter an already-established originating-client tool
  route. Replay observes/forwards the session but does not own provider arbitration.
- Pi HTTP provider re-entry during an active ACP provider prompt remains transparent and
  cannot recursively create another ACP delivery.

## Verification

- `pytest`: 57 passed
- Python compilation: clean
- No Python source/test lines exceed the configured Ruff 100-character line length.

Local Ruff and mypy execution remains required in the project development environment.
