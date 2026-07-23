# Lumen v3.0.0

Lumen v3 introduces bounded-session continuity: a context window may end, but the work does not.

## Implemented

- `\\obt` control-plane commands intercepted at the start of user messages and never forwarded to the model.
- One-time session banner explaining the command namespace.
- Rolling model-authored summarisation begins at 75% context usage.
- Preferred working ceiling recorded as 80%; hard continuation boundary at 95%.
- At the hard boundary Lumen writes an append-only recovery journal and returns a structured `lumen-continuation` payload rather than invoking the model.
- Inherited checkpoint summaries remain separate, ordered system messages and are not merged into later summaries.
- Explicit `continuation_required` lifecycle outcome distinguishes safe boundary completion from runtime failure.
- `\\obt session continuation` exposes the latest in-memory recovery payload.

## Chapter Principle

A Lumen session is a chapter: it preserves the understanding needed for the next chapter rather than every word. Source remains authoritative and may be re-read whenever exact details matter.
