# Selected Experimental Investigations

**Status:** Curated navigation index  
**Reviewed:** 2026-11-04  
**Source:** [Lumen Engineering Diary](../engineering/ENGINEERING_DIARY.md) (chronological, append-only engineering record)

This page is an entry point for external researchers and engineering reviewers. It highlights a small number of documented investigations and preserves the distinction between **observed results**, **interpretation**, and **unanswered questions**. The diary contains the fuller methods, context, failures and subsequent decisions; this index is not a peer-reviewed publication or an independent replication report.

## 1. Long-context source analysis and continuity — 20–23 July 2026

**Question:** Can a Qwen model operating within a bounded context window complete a reading task involving a roughly 10,000-line production source file through rolling continuity checkpoints?

**Method:** Repeated source reading with checkpoints/context reconstruction. The diary records an intended controlled comparison using the same task, prompt, source file, model (Qwen 14B) and reading strategy between versions.

**Observed:** The source-reading task completed; checkpoints maintained task continuity. Later analysis found that function inventories developed more strongly than higher-level architectural synthesis, and that the final answer did not necessarily reflect the richness of checkpoint material.

**Limitations:** This is a task-specific engineering demonstration, not evidence that a model's true internal cognitive state was measured or that answer quality universally improved.

**Diary navigation:** Sections **“Reading Beyond the Native Context Window”**, **“v3.2.4 Experimental Evaluation”**, **“v3.2.5 Experimental Results”** and **“v3.2.6 Development Diary”**.

## 2. Evidence-linked checkpoint prompt: unsuccessful hypothesis — 24 July 2026

**Question:** Would requiring each checkpoint conclusion to include observation, source evidence, confidence and uncertainty improve retained architectural understanding?

**Method:** Version 3.2.7 changed one checkpoint section and compared its behaviour with the previous benchmark.

**Observed:** The revised section remained generic and the resulting architectural representations were less detailed than in the comparison run. The diary records that the hypothesis was **not validated** and that the experimental prompt change was reverted.

**Limitations:** The run was interrupted at approximately line 6,753, so the result is an initial comparative observation rather than a complete controlled trial. Other operational issues were recorded separately.

**Why it matters:** It is a concrete example of documenting a negative result and changing direction in response to evidence.

**Diary navigation:** Section **“Lumen v3.2.7 Development Diary — Evaluation concluded”**.

## 3. Checkpoint quality versus final-answer quality — 25 July 2026

**Question:** Does a richer retained architectural representation necessarily produce a stronger final response?

**Observed:** Version 3.2.8 produced substantially richer checkpoints, while the user-facing answer remained comparatively generic. The experiment also surfaced end-of-file handling, an unexpected refusal and result-persistence defects.

**Interpretation:** The diary distinguishes retained checkpoint content from final response generation and identifies the completion pipeline as an additional source of failure.

**Limitations:** A checkpoint is generated model text, not a direct measurement of hidden model reasoning. The unexpected refusal's cause was not established in the documented evaluation.

**Diary navigation:** Section **“Lumen v3.2.8 – Evaluation Summary”**.

## 4. Model tool calls, dispatch and procedural compliance — 27–31 October 2026

**Question:** How can an evaluation distinguish a model's stated intention, an advertised tool, a requested tool, actual tool execution and the final filesystem result?

**Method:** Tool-routing investigations, controlled timeout comparison, Qwen tool-call normalisation and file-editing tasks checked against operational evidence.

**Observed:** A timeout change resolved one reproduced detach scenario; later tests recorded real SDK tool calls. In a file-editing scenario, the final file content matched the requested outcome, while the tool-call sequence did not fully follow the required standalone ordering.

**Interpretation:** **Task outcome succeeded; procedural instruction compliance was imperfect.** Neither a model's success claim nor a successful transport response alone proves the full task requirements were met.

**Limitations:** The timeout finding does not establish the cause of every detach; successful runs do not certify all failure paths or tool schemas.

**Diary navigation:** Sections **“Pi Tool Routing and the First End-to-End Tool Cycles”**, **“Qwen Specialised Adapter: Mixed Commentary and Tool Calls”** and **“File-Editing Tests and the Evidence/Outcome Distinction”**.

## 5. Reproducible execution planning — October 2026 onward

**Question:** What execution configuration must be preserved to support meaningful replay of a model interaction?

**Engineering direction:** Session-owned model reservation, a resolved immutable Session Execution Plan (SEP), evidence acknowledgement, and separate Original Setup/New Setup replay contracts.

**Status:** Architecture and substantial implementation/integration work are documented. Full replay alignment and formal cross-service verification remained outstanding in the 31 October roadmap. **This is an infrastructure investigation, not a completed behavioural study.**

**Diary navigation:** Sections **“Session-Owned EMD/SEP Lifecycle”**, **“Execution Ownership and Observability”** and **“Consolidated Requirements and the NEXT Development Sequence”**.

## Reading the Evidence

Start with the [Engineering Diary](../engineering/ENGINEERING_DIARY.md) for the chronological record and [Current Status](../CURRENT_STATUS.md) for the present development boundary. See the [Canonical Architecture](../architecture/core/ARCHITECTURE.md) for the current implementation model. Earlier experiments may describe superseded architectures; their observations are retained for provenance rather than silently updated.
