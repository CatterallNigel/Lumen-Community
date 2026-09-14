# Lumen M0.1 Replay Runtime Artifacts — 2026-09-07

**Status:** Recorded during Phase 9 investigation  
**Disposition:** Repetere findings are covered by the planned Experiment/Run work. The Moderari/Praebere bypass is documented but is not to be investigated separately for M0.1.

## Test outcome

Two executions of the same staged Replay completed successfully and produced a full match. The second execution occurred without intervening ordinary Lumen model traffic.

## 1. Incorrect Repetere operation banner

The Repetere UI displayed the red banner:

> Operation failed: replay running.

This message was present while the Replay was running and remained after the Run completed successfully. `RUNNING` is an accepted lifecycle state, not an operation failure. Terminal completion must replace or clear transient start-state messaging. This is active Repetere Phase 9 UI work.

## 2. Replay session not closed and context reused

Repetere did not close its Pontis session after the first successful Run. The second Run reused the same session identifier.

The operational log confirms that Moderari retained and reconnected the earlier context:

- first Run completed and persisted a terminal result for the session;
- second Run reused the same session ID;
- Moderari logged `client_session_context_reconnected`;
- four prior messages were restored and combined with two incoming messages;
- the second Run therefore executed with six effective messages rather than an isolated fresh context.

This confirms that every Run must receive a fresh Pontis session and that Repetere must close that session on every terminal path. Cleanup must also release only that Run's Praebere reservation contribution.

## 3. Model activated outside Praebere lifecycle

Clean-start testing established:

- Ollama initially had no model resident;
- Praebere reported no selected model, no reservations and no active executions;
- Repetere sent the recorded request through Moderari with `qwen2.5-coder:14b-32k` in the request;
- Ollama loaded that model in response to the request;
- Praebere continued to report that the model was neither selected nor reserved through Praebere, while recognising it as externally resident;
- the Replay nevertheless completed successfully.

This demonstrates that a model-bearing request reaching Moderari/Ollama can cause Ollama to activate a model without Praebere selection, reservation or lifecycle ownership.

### Classification

Record this as a possible Moderari model-lifecycle bypass artifact. Do not investigate or correct it as a separate task during the current M0.1 Phase 9 work.

The agreed Repetere prerequisite flow will prevent the path operationally:

1. read the required provider/model from the source Trace;
2. create a new Run and fresh Pontis session;
3. query authoritative Praebere state through Nuntius;
4. validate availability and reject any conflicting selected, reserved, resident or locked model;
5. select and reserve the required model through Praebere;
6. explicitly activate it through Praebere and require positive acknowledgement;
7. establish Moderari Pass-through;
8. start the Replay child Trace;
9. execute the Replay;
10. close the Pontis session and release the Run's reservation on every terminal path.

Once this flow is enforced, Repetere will never intentionally send a Replay model request to Moderari before Praebere has authorised and activated the required model.

## Evidence

- Repetere UI screenshot showing the red failure banner alongside a successfully matched Experiment.
- Pontis UI showing the Replay session remained open after completion.
- Moderari UI showing two terminal-result generations retained for the reused session.
- Praebere UI showing no selected model or reservations after Ollama loaded the requested model externally.
- Servire operational log: `lumen-servire-operational-2026-09-07T20-01-44Z-Second-Replay-After-First.log`.

