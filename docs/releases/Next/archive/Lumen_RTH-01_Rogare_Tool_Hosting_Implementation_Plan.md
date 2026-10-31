# Lumen — RTH-01 Rogare Tool Hosting Implementation Plan

**Status:** Approved architectural direction; implementation pending  
**Branch:** `research-foundation-next`  
**Development:** Windows / Python 3.12 / Node.js  
**Deployment target:** Single Lumen Linux application image (packaging deferred to release preparation)  
**Date:** 2026-10-29

## 1. Purpose

Replace the current Pi ACP agent-mediated tool execution path with direct use of Pi SDK tool implementations hosted by **Rogare**. **Pontis remains the Client/Tool Bridge**, transporting tool requests and results; **Moderari** remains responsible for SEP-governed orchestration and model continuation; **Praebere** remains authoritative for provider/model execution.

The immediate acceptance scenario is a Rogare Ask that creates `lumen-tool-test.txt`, reads it back, and reports the exact contents, without a second model conversation or a silent session closure.

## 2. Decisions and boundaries

1. **Rogare is the initial tool host.** It executes the four initial tools: `read`, `write`, `edit`, `bash`.
2. **Use the Pi SDK directly.** Do not invoke `pi-acp`, Pi agent prompts, or Pi model inference to execute tools. Verify the pinned SDK's actual exports and execution signatures before coding the adapter.
3. **No web-search tool.** Do not load Pi extensions or expose other tools by default. The initial tool catalogue is an explicit four-tool allowlist; no Ollama-site login or other external credentials are required for tool execution.
4. **Pontis is tool-host neutral.** It relays structured tool calls and results, correlates them with the Lumen session, and owns session termination coordination. It does not execute the four tools itself.
5. **Moderari controls permission and continuation.** The selected Filter Profile and materialized SEP determine which tools may be called. Tool results continue the *same* Ask and model conversation.
6. **Praebere remains the only model execution path.** Pi's reported/default model is irrelevant to Lumen execution.
7. **ACP/MCP external tool connections are deferred.** When introduced, they connect through Pontis. Remote-client tool-location policy is a separate future design decision.
8. **Docker packaging is deferred.** Maintain Linux compatibility now, but do not require a new M0.1 image to complete RTH-01.
9. **Existing Pi ACP implementation remains available only during migration**, then is retired from this tool path after acceptance tests pass.

## 3. Target flow

```text
User
  |
  v
Rogare -- Ask --> Pontis --> Moderari --> Praebere --> Model
  ^                  ^           |
  |                  |           | structured tool call (SEP-authorized)
  |                  +-----------+
  |                  |
  +-- tool request --+  (Pontis routes to Rogare tool host)
  |
  +-- Pi SDK tool implementation: read / write / edit / bash
  |
  +-- structured tool result --> Pontis --> Moderari
                                            |
                                            +--> Praebere --> Model continuation
                                                                  |
Rogare <-- final answer <-- Pontis <-- Moderari <----------------+
```

**Invariant:** A tool invocation and its result are subordinate to the originating Ask, not a new Ask or a new provider/model conversation. Pontis must not forward Pi-generated provider re-entry requests as a substitute for tool execution.

## 4. Interface contract (proposed; finalize against current service code)

### 4.1 Tool request

```json
{
  "protocol_version": "1",
  "session_id": "...",
  "execution_id": "...",
  "tool_call_id": "...",
  "tool_name": "write",
  "arguments": {"path": "lumen-tool-test.txt", "content": "..."},
  "workspace_id": "...",
  "timeout_ms": 30000
}
```

### 4.2 Tool result

```json
{
  "protocol_version": "1",
  "session_id": "...",
  "execution_id": "...",
  "tool_call_id": "...",
  "status": "success",
  "content": [{"type": "text", "text": "..."}],
  "error": null
}
```

Final schemas must be based on the existing Moderari/Pontis continuation and Rogare transport contracts. Preserve the original model `tool_call_id`; distinguish transport failure, tool execution failure, timeout, cancellation, and successful output. Ensure any content blocks, structured metadata, and exit codes needed by the model are retained. Define request/result size limits.

### 4.3 Tool execution safety

- Validate tool name and arguments against the pinned SDK's actual tool schema.
- Resolve paths against a configured workspace; prevent unintended traversal and symlink escape for file tools.
- **Do not treat the workspace as a shell sandbox.** `bash` can access broader filesystem/process resources; implement explicit enablement, timeouts, process-tree termination, and deployment-specific privilege controls. Do not claim isolation without testing it.
- Define session cancellation behavior and duplicate-result handling. Side-effecting operations such as `write` and `bash` must not be blindly retried.
- Do not expose Pi authentication or model configuration files to the tool host. Tool-only SDK initialization must work without them.

## 5. Implementation increments

### RTH-01.0 — Contract and SDK verification

**Owner:** Cross-service design / Rogare

- Inspect current Rogare, Pontis, and Moderari source to identify existing transport, tool-call parsing, pending-call state, and continuation hooks.
- Verify Pi SDK package/version, exact exports, execution signatures, tool schemas, result types, dependencies, and licence.
- Prototype standalone `read`, `write`, `edit`, `bash` calls without `pi-acp`, Pi model configuration, Pi authentication files, or provider requests.
- Verify Windows shell requirements for `bash` and establish Linux-equivalent behavior.
- Freeze a minimal versioned tool-request/result contract; avoid creating a parallel Moderari orchestration mechanism.

**Exit:** Four standalone SDK tools work; precise cross-service contract and ownership are documented.

### RTH-01.1 — Rogare SDK tool host

**Owner:** Rogare

- Add a small, independently testable Node.js SDK adapter and a Rogare-side tool-host integration.
- Load only `read`, `write`, `edit`, `bash` by explicit allowlist.
- Implement validated dispatch, configured workspace, timeouts, cancellation, normalized results/errors, and session/execution correlation.
- Ensure no Pi ACP process or model/authentication configuration is required.
- Provide unit tests for success, missing file, invalid arguments, rejected paths, timeout, disabled tool, and execution errors.

**Exit:** Rogare can execute each tool locally from a structured request and return a structured result.

### RTH-01.2 — Pontis tool relay

**Owner:** Pontis

- Replace the `session_service.deliver(..., call.fork_message, ...)` **agent-prompt execution path** for these tools with a request to the Rogare tool host.
- Relay the original `session_id`, `execution_id`, `tool_call_id`, tool name and arguments unchanged in meaning.
- Handle transport errors, disconnects, multiple calls, cancellation and bounded waits.
- Ensure Pi provider re-entry cannot be mistaken for the tool result; remove the old path only after tests pass.
- Keep routing interfaces suitable for future external ACP/MCP hosts through Pontis, without implementing those hosts now.

**Exit:** Pontis routes and correlates tool requests/results without initiating a second model conversation.

### RTH-01.3 — Moderari SEP and continuation alignment

**Owner:** Moderari, with Pontis integration

- Reuse the existing SEP tool-permission and execution machinery where possible.
- Validate that the requested tool is permitted for the session and current execution.
- Track pending tool calls; accept only matching results and reject stale, duplicate or unsolicited results.
- Append model-compatible tool results to the same conversation, then resume via Praebere.
- Support sequential `write` then `read`, and multiple tool calls where the model requests them.
- Surface tool failure to the model/user rather than returning a blank successful answer.
- Preserve streaming/non-streaming behavior and existing no-tools paths.

**Exit:** One Ask can complete model → tool → model continuation, with no duplicate inference thread.

### RTH-01.4 — Pontis forced-close notification

**Owner:** Pontis and Rogare

- Before force-closing an active session, send Rogare a structured terminal-failure notification with session ID, reason, and correlation details.
- Require an explicit Rogare acknowledgement of receipt before closing, with a **bounded timeout and documented fallback** if Rogare is unreachable.
- Ensure Rogare displays the failure before losing the session and does not present an empty answer as success.
- Preserve Pontis authority for termination and notifications to Moderari/Praebere and other relevant services.
- Test ordering, acknowledgement, timeout, duplicate notifications and disconnected clients.

**Exit:** No silent force close in a reachable Rogare client; cleanup cannot hang indefinitely.

### RTH-01.5 — Windows integration and regression verification

**Owner:** Cross-service

- Run unit and contract tests in each modified repository.
- Run `pytest`, `ruff check .`, and `mypy src tests` for modified Python services; run Node.js type/lint/test checks appropriate to the chosen adapter.
- Confirm normal Ask behavior without tools, and Filter Profile `None` / disabled-tools pass-through behavior.
- Execute the exact end-to-end test:

```text
Using your available tools, create a file named lumen-tool-test.txt in the current working directory.

Write exactly these three lines:

Lumen Tool Integration Test
Tool: Pi ACP
Status: SUCCESS

Then use the read tool to read the file back and report its exact contents.

Do not claim success unless both tool operations complete successfully.
```

The test's `Tool: Pi ACP` line is **literal file content from the original regression scenario**, not an architectural requirement. A follow-up SDK-specific acceptance test may use `Tool: Pi SDK`.

- Verify file bytes, tool-call/result correlation, final response, trace evidence, absence of Pi-originated provider re-entry, and no unintended session closure.
- Run negative tests: unauthorized tool, malformed args, unavailable host, tool failure, timeout, duplicate result and force close.

**Exit:** Full Windows end-to-end regression succeeds and the changed services pass their quality gates.

### RTH-01.6 — Docker release packaging (**deferred**)

**Owner:** M0.1 distribution

- Update the final Linux image to include the pinned SDK/Node.js runtime and Rogare tool-host adapter.
- Retire obsolete `pi-acp` runtime/config/auth dependencies **only if no other component requires them**.
- Validate `/workspace` persistence, non-root permissions, process lifecycle, Linux shell behavior, filesystem boundaries and resource limits.
- Run the same end-to-end and failure tests in the release image.

**Not a blocker for RTH-01.0–01.5.**

## 6. Source references and deployment considerations

The supplied M0.1 files are **reference material**, not an instruction to build an image now:

- `README(20261029-162303).md` describes an earlier transitional Pontis + Pi/pi-acp container and explicitly notes that Pi versions were not pinned in that foundation.
- `Dockerfile(6)` is the current Docker build reference and should be rechecked when packaging begins.
- `compose(10).yaml` defines a Lumen application container, a separate MongoDB service, HAProxy and a persistent `lumen-workspace` volume mounted at `/workspace`.

Do not bake credentials into the future release image. The current Compose reference includes a plaintext MongoDB administrator credential; rotate it if active and replace it with an appropriate secret-injection mechanism before distribution.

## 7. Acceptance checklist

- [ ] SDK version, exports and licence verified against actual package.
- [ ] All four SDK tools execute without Pi ACP, Pi model configuration, Pi authentication files or Pi model inference.
- [ ] Rogare owns initial tool execution; Pontis only relays.
- [ ] Tool availability is constrained by SEP and the explicit four-tool allowlist.
- [ ] Original session/execution/tool-call identity is preserved end-to-end.
- [ ] Tool results resume the original Ask through Moderari and Praebere.
- [ ] `write` → `read` → final model response succeeds, with verified file contents.
- [ ] Tool failures and transport failures are visible and do not masquerade as success.
- [ ] Pontis notifies and receives Rogare acknowledgement before force close, subject to bounded fallback.
- [ ] Existing non-tool and tools-disabled conversations continue to work.
- [ ] Windows automated tests and static checks pass.
- [ ] Linux deployment requirements documented; actual Docker integration deferred.

## 8. Out of scope

- Web search or other Pi extensions.
- Pi ACP agent-driven tool execution.
- New model selection, reservation or EMD behavior.
- External ACP/MCP tool hosts, remote-client execution and tool-host discovery.
- Docker image changes and release distribution in this increment.

## 9. First implementation action

Begin with **RTH-01.0**: inspect the current Rogare/Pontis/Moderari interfaces and verify the Pi SDK's exact tool exports and standalone execution behavior. Only then freeze the transport schema and implement Rogare's tool host. This avoids embedding assumptions about Pi's APIs or duplicating existing Moderari orchestration.
