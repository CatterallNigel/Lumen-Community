# Lumen Pontis M6.1 — Rogare Session Context Compatibility

## Objective

Allow Rogare-originated OpenAI HTTP conversations to carry stable Lumen session and working-directory context into the existing Pontis M6 tool bridge, without changing or breaking the provider re-entry path used by Replay.

## Changes

- Pontis continues to resolve `x-lumen-session-id` as the preferred logical continuation/session identifier.
- Pontis continues to resolve `x-lumen-cwd` as provider-session working-directory context.
- Rogare can therefore use the existing M6 lazy provider-session mechanism without a Rogare-specific tool bridge.
- `GET /_pontis/provider-capabilities` reports that provider execution bridging exists while generic model-tool-catalogue discovery is not currently supported.
- No tool names or schemas are invented or inferred.

## Protected compatibility invariant

The M6 provider re-entry rule remains authoritative. If Pi is already processing an ACP prompt and re-enters Pontis over HTTP, Pontis forwards that HTTP request downstream once and returns the downstream response transparently. It must not recursively create another ACP/provider cycle. This is the path Replay depends on after a fork.

## Automated acceptance

- complete Pontis test suite passes;
- provider re-entry regression includes Lumen session/CWD headers and still performs one downstream pass only;
- lazy provider-session reuse remains covered;
- capability diagnostic explicitly reports discovery as unsupported.
