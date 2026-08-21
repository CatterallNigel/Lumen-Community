# Lumen Pontis M6.3 — Provider Capability Arbitration

## Objective

Keep client/tool-provider arbitration inside Pontis without making Pi part of Rogare's ongoing
conversation transport. Reuse the already-proven ACP→HTTP provider re-entry to learn the provider
contract, then keep subsequent Rogare turns on the normal direct HTTP Lumen path.

## Implemented

- During explicit `\obt rogare start`, mark the stable Lumen session as awaiting a provider
  capability declaration.
- Observe Pi's normal HTTP re-entry while preserving the unchanged M6 transparent provider
  re-entry path.
- Capture the provider-selected model, exact OpenAI-compatible tool declarations and `tool_choice`
  when present.
- Bind that capability snapshot to the Lumen session rather than to Rogare or Moderari.
- Enrich later requests carrying that bound session ID with the provider-controlled model/tool
  contract before they enter Vestigare → Repetere → Moderari.
- Preserve client-owned messages and stream choice.
- Continue routing actual model tool calls through the existing bound ACP provider session.
- Clear provider capabilities when the Lumen/ACP session is closed or Pontis shuts down.
- Report capability-discovery support through `GET /_pontis/provider-capabilities`.
- Leave sessions without a snapshot unchanged.

## Architectural result

```text
Rogare / other client
        ↓
      Pontis  ←→ ACP tool provider
        ↓
   Vestigare
        ↓
   Repetere
        ↓
   Moderari
        ↓
      Model
```

Pontis owns client/session/provider/capability mediation. The rest of the Lumen stack sees Pontis,
not the concrete client or tool provider. Pi remains a tool provider and is not required as an
ongoing conversational bridge.

## Replay compatibility invariant

The provider re-entry branch remains first-priority while an ACP prompt is active. Replay-originated
ACP→HTTP traffic is still forwarded exactly once downstream and is not recursively fed into the ACP
tool bridge. Capability capture is only armed by an explicit Rogare bootstrap session, so ordinary
Replay sessions do not acquire or overwrite provider snapshots.

## Automated verification in delivered package

- `pytest`: 52 passed.
- Python compile check: passed.
- Added provider snapshot capture and bound-client enrichment coverage.
- Existing Replay-safe provider re-entry regression coverage remains passing.
- Ruff/mypy could not be executed in the packaging environment because the executables are not
  installed and package download is unavailable; run both in the normal local development
  environment.

## Live acceptance test

1. Start the Lumen stack and Pontis.
2. Start Rogare session `rogare-test-1` using `\obt rogare start`.
3. Confirm the returned control payload reports `capabilities_ready=true`, the captured model, and a
   non-zero tool count.
4. Send a normal Rogare message on the same session that requires `bash`, such as `pwd`.
5. Confirm Pontis logs `Pontis enriched bound client request` with the captured model/tool count.
6. Confirm Moderari receives the real model name and Pi tool declarations rather than
   `model=lumen tools=[]`.
7. Confirm the model emits the tool request and Pontis reuses the existing ACP binding
   (`provider_session_created=false`).
8. Confirm the final answer returns through Pontis to Rogare.
9. Re-run a known Replay fork/tool case and confirm the existing ACP→HTTP provider re-entry behavior
   remains unchanged.
