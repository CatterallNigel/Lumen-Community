# Lumen v3.2.0

## Cumulative Distilled Continuity

Lumen now asks Qwen to maintain one cumulative engineering understanding rather than producing isolated summaries of retired source chunks.

Each Continuity Checkpoint request contains:

- the user's original objective;
- the previous Distilled Continuity;
- deterministic source coverage for the newly retired material;
- the newly read raw source material;
- a short instruction to preserve, extend, connect, or correct the existing understanding.

The prompt explicitly tells Qwen not to describe the new chunk in isolation.

Only the latest cumulative Distilled Continuity is injected into the active model context. Every prior Continuity Checkpoint remains archived immutably for later audit and comparison.

## Source Coverage

Each Continuity Checkpoint now records source ranges containing:

- file path;
- start offset;
- end offset.

Lumen derives these values from the actual read tool calls and results rather than asking Qwen to reproduce them. Adjacent and overlapping ranges for the same file are merged into cumulative Source Coverage.

This provides the foundation for targeted rereads, coverage-gap detection, cross-file understanding, and checkpoint observability.

## Terminology

User-visible checkpoint progress now uses the Lumen lexicon:

- Continuity Checkpoint
- Distilled Continuity
- Working State
- Source Coverage

Legacy internal Python function names remain temporarily available for compatibility.

## Validation

109 automated tests pass.
