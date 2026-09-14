# Lumen Repetere — Definitive Matched and Divergent Replay Behaviour

**Status:** Authoritative implementation reference  
**Scope:** Lumen M0.1 Replay lifecycle  
**Agreed:** 9 September 2026

## 1. Purpose

This document defines the required behaviour of Repetere for a non-divergent Replay and a divergent Replay. It is the definitive reference for the remaining implementation work.

The central distinction is:

- A non-divergent Replay remains private. Repetere replays and compares the recorded interaction without sending model traffic to Pi.
- A divergent Replay records the first behavioural difference and then changes into a transparent proxy. The actual divergent response is allowed to continue through the prepared Pontis/Pi tool path until the complete ask-to-answer interaction ends.

A divergence is a comparison result. It is not an instruction to terminate the ask before its final answer.

## 2. Service ownership

### Repetere owns

- The Replay Experiment and Run.
- The source Trace and replay sequence.
- Comparison of live responses with recorded responses.
- First-divergence detection and evidence.
- Injection of recorded tool results while the Replay continues to match.
- The transition to transparent proxy behaviour after divergence.
- Starting and stopping the child Vestigare recording.
- Requesting creation and closure of its Pontis session.
- Persisting the terminal Run outcome.

Repetere knows its Pontis session ID. It does not need to know Pontis's internal ACP session ID or ACP bookkeeping.

### Vestigare owns

- Recording the child Replay Trace.
- Binding the recording to the Repetere session, Experiment, Run and source Trace.
- Recording privately ingested matching exchanges.
- Transparently recording live traffic after divergence.
- Completing the recording when instructed by Repetere.

Vestigare does not decide whether a Replay matched, diverged or completed.

### Pontis owns

- The authoritative Lumen session.
- Establishing and maintaining the Pi ACP connection associated with that session.
- Associating Pi's HTTP provider traffic with the correct Lumen session.
- Coordinating model reservation and activation through Praebere.
- Detecting the ACP `end_turn` outcome after a divergent live continuation.
- Notifying Repetere that the divergent Replay conversation has completed.
- Closing its session and internal ACP resources when Repetere requests session closure.
- Informing Praebere that the session has terminated.

Pontis does not decide the Replay result and does not stop Vestigare.

### Praebere owns

- Model selection, reservation and activation.
- Loading the selected model when required.
- Tracking every session using the model.
- Releasing a terminated session's reservation and execution.
- Unloading the model only when no other session still requires it.

### External Pi tool provider

Pi is an external client and tool provider. Its responsibility is limited to:

- Executing tool requests it receives.
- Returning the resulting tool output through its normal HTTP provider request.

Pi does not own the Replay, its continuation, its comparison state, its completion decision or the Lumen session lifecycle.

## 3. Common Replay preparation

Both Replay outcomes begin with the same preparation:

1. Repetere creates a new Run for the staged Experiment.
2. Repetere requests a fresh Pontis session with origin `repetere`.
3. Pontis creates the authoritative Lumen session.
4. Pontis establishes and binds the corresponding Pi ACP connection without requiring Repetere to submit a conversational ask to Pontis.
5. Pontis associates Pi's HTTP provider traffic with the Repetere session.
6. Pontis coordinates reservation and activation of the recorded model through Praebere.
7. Praebere loads the model if it is not already resident.
8. Pontis confirms that the Repetere session is ready.
9. Repetere starts a child Vestigare Trace bound to the session, Experiment, Run and source Trace.

The ACP/Pi route is prepared at this stage but remains dormant during a completely matching Replay. No canned prompt or additional conversational ask may be used merely to create the ACP connection because that would alter context and contaminate the child Trace.

## 4. Non-divergent Replay

A non-divergent Replay is private. No model response or tool request is passed to Pi.

1. Repetere constructs the first live model request from the recorded source exchange.
2. Repetere sends the request directly to Moderari using the fresh Replay session identity.
3. Moderari calls the model and returns the live response to Repetere.
4. Repetere compares the live response with the corresponding recorded response.
5. Repetere records the live request and response in the child Trace through Vestigare's private recording-ingestion path.
6. If the matched response contains a tool request:
   - Repetere does not pass the tool request to Pi.
   - Pi does not execute the tool.
   - Repetere takes the recorded tool result from the source Trace.
   - Repetere injects that recorded tool result into the next live model request.
7. Repetere repeats the private request, response, comparison and recording cycle for every recorded exchange.
8. The final live response matches the recorded final response.
9. Repetere confirms that all meaningful source exchanges have been reproduced and matched.
10. Repetere sets the Run result to `MATCHED`.
11. Repetere tells Vestigare to stop recording.
12. Vestigare completes and persists the child Trace.
13. Repetere tells Pontis to close the Replay session.
14. Pontis closes the Lumen session and its internal ACP resources.
15. Pontis tells Praebere that the session has terminated.
16. Praebere releases the session's model execution and reservation.
17. Praebere unloads the model if no other session requires it.
18. Repetere persists the terminal Run state as `MATCHED · COMPLETED`.

During a completely matching Replay:

- Pi receives no model traffic.
- Pontis carries no Replay model traffic.
- No live tool is executed.
- Recorded tool results provide deterministic continuation.

Repetere determines completion because it owns the source sequence and knows when every meaningful recorded exchange has matched.

## 5. Divergent Replay

A divergent Replay begins privately and follows the non-divergent sequence until the first meaningful mismatch.

1. Repetere constructs and sends recorded exchanges privately to Moderari.
2. Repetere compares and privately records each live response.
3. Recorded tool results continue to be injected while the interaction matches.
4. Repetere detects the first meaningful difference between a live response and its recorded counterpart.
5. Repetere persists the first-fork evidence, including the expected and observed behaviour.
6. Repetere marks the Run as divergent but still running.
7. Repetere stops comparison and stops injecting recorded continuation data.
8. The actual divergent model response is passed unchanged into the already-prepared route:

   `Repetere -> Vestigare -> Pontis -> Pi`

9. Repetere becomes a transparent proxy. It does not interpret, alter or compare subsequent conversation messages.
10. Vestigare transparently records the live divergent continuation.
11. If the divergent response requests a tool, Pi executes the real tool.
12. Pi returns the tool result in its next HTTP provider request through:

   `Pi -> Pontis -> Vestigare -> Repetere -> Moderari`

13. The model response returns through:

   `Moderari -> Repetere -> Vestigare -> Pontis -> Pi`

14. The standard request path may contain any number of further model responses and Pi tool executions.
15. Repetere and Vestigare remain transparent throughout this continuation; Vestigare continues recording it.
16. When the complete interaction ends, Pontis detects ACP `stop_reason=end_turn`.
17. Pontis sends Repetere a session-scoped notification that the Replay conversation has completed.
18. Repetere sets the Run result to `DIVERGED`.
19. Repetere tells Vestigare to stop recording.
20. Vestigare completes and persists the full child Trace, including the complete divergent result.
21. Repetere tells Pontis to close the Replay session.
22. Pontis closes the Lumen session and its internal ACP resources.
23. Pontis tells Praebere that the session has terminated.
24. Praebere releases the session's model execution and reservation.
25. Praebere unloads the model if no other session requires it.
26. Repetere persists the terminal Run state as `DIVERGED · COMPLETED`.

The first divergence is retained as comparison evidence, while the entire subsequent interaction is retained as the divergent result.

## 6. Conversation-completion authority

An individual HTTP or SSE response completing does not necessarily mean that the conversation has completed. A response may request one or more tools; after Pi returns the tool output, the model interaction continues through another provider request.

The relevant signals are:

| Signal | Meaning | Whole divergent conversation complete? |
| --- | --- | --- |
| HTTP/SSE response ends | One model invocation completed | No |
| Model `finish_reason: tool_calls` | Pi must execute one or more tools | No |
| ACP tool result completed | One tool execution completed | No |
| Another provider request carries a Pi tool result | The interaction is continuing | No |
| Model `finish_reason: stop` | The model produced a response requiring no tool | Supporting evidence only |
| ACP `stop_reason=end_turn` | The complete interaction ended | Yes |

Pontis owns ACP and therefore owns detection of `end_turn`. Repetere does not inspect or validate ACP internals. It receives only the session-scoped completion notification required to finish its Run.

## 7. Cleanup rules

Cleanup preserves service ownership:

1. Repetere determines that the matched Replay is complete, or receives Pontis's completion notification for a divergent Replay.
2. Repetere stops Vestigare.
3. Repetere asks Pontis to close its session.
4. Pontis closes its own Lumen and ACP resources.
5. Pontis informs Praebere that the session has terminated.
6. Praebere releases the model execution and reservation.
7. Praebere decides whether the model can be unloaded.

Pontis detecting `end_turn` does not itself start cleanup. It notifies Repetere. Repetere then initiates the ordered shutdown.

Cleanup operations must ultimately be idempotent so that ordinary completion, cancellation, timeout and recovery cannot leave Vestigare recording, Moderari waiting, a Pontis session open or a Praebere execution reserved indefinitely.

## 8. Required terminal outcomes

- Full comparison with no divergence: `MATCHED · COMPLETED`.
- First divergence followed by a complete transparent continuation: `DIVERGED · COMPLETED`.
- A divergence alone is not completion; the live continuation and final answer must be recorded first.
- A stalled or failed continuation must not be reported as successfully completed and must enter an explicit failed, timed-out or cancelled terminal state after bounded recovery and cleanup.

## 9. Implementation invariant

The definitive invariant is:

> Repetere privately reproduces and compares recorded behaviour while it matches. At the first divergence, it preserves the fork and becomes transparent so the real ask can continue through its tools and final answer. Pi executes requested tools, Vestigare records the evidence, Pontis owns the session and ACP boundary, and Praebere owns the model lifecycle.
