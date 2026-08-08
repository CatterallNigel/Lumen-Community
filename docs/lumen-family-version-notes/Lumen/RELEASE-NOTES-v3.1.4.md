# Lumen v3.1.4

## Persistent resumed-session base context

- A staged MongoDB restore is bound to the active Pi session after the first ordinary prompt.
- Restored summaries, objective history and retained raw context are reapplied on every subsequent model/tool round.
- The restored base is injected once per outbound request and is not appended repeatedly to Pi's own history.
- A newly selected restore replaces any previous active restored base for that session key.
- Initial activation logs `restore_applied`; later rounds log `resumed_context_applied`.
