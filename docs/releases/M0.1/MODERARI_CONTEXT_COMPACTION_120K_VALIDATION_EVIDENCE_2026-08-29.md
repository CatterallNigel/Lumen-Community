# Lumen M0.1 Validation Evidence Record

## Moderari Rolling Context Compaction --- 120K Log File

**Test date:** 29 August 2026\
**Evidence reviewed:** 30 August 2026\
**Status:** Completed execution; compaction mechanism validated with an
important behavioural finding\
**Model:** `qwen2.5-coder:14b-32k`\
**Client:** Pi\
**Session:** `session-http-edd3ed22-7099-4470-bfd1-14dfc050034b`\
**Trace recording:**
`Compaction-Test-Saturday-2_20260829_155746_6002212a`\
**User ask:** `List the files in this directory`\
**Directory:** `C:/Development/Lumen-Test-Data`\
**Discovered file:** `logfile_120K.log`

------------------------------------------------------------------------

## 1. Purpose

This test was intended to exercise Moderari's rolling context-compaction
mechanism using a sufficiently large source file and a local 32K-context
model.

The execution developed into a more useful behavioural experiment than
originally intended. The user's explicit request required only directory
enumeration. Qwen correctly executed `ls -1`, which satisfied that
request, but then independently began reading the discovered
`logfile_120K.log` in successive chunks.

No intervention was made. The continuation was retained as observable
model behaviour rather than treated as an error to be prevented.

The resulting run therefore exercised:

-   autonomous model/tool behaviour;
-   long-running session continuity;
-   repeated context growth;
-   two successive rolling compactions;
-   continuity checkpoint persistence and reinjection;
-   source-range tracking;
-   preservation of the original user objective across compaction;
-   behaviour after the checkpoint explicitly declared the task
    complete; and
-   terminal checkpoint creation.

------------------------------------------------------------------------

## 2. Evidence Sources

The record was reconstructed from:

1.  `lumen.trace_messages-Compaction-120k-File.json`
    -   Mongo/Vestigare trace-message export.
    -   Provides the Pi → Lumen requests and Lumen → Pi responses,
        effective system-prompt provenance, model-visible message
        history, and tool-call sequence.
2.  `Moderari-Compact-120k-File.zip`
    -   `moderari.log`
    -   `interaction.log`
    -   `protocol.jsonl`
    -   `ui.log`
    -   `audit.log`
3.  Pi terminal output captured during the test.

Times below use the local timestamps emitted by Moderari where
available. Mongo trace timestamps are UTC.

------------------------------------------------------------------------

## 3. Initial Execution

The first Pi request contained the user instruction:

> `List the files in this directory`

Moderari operated under its `default` system-prompt policy and replaced
Pi's incoming system prompt with the generated Moderari prompt.

The effective prompt included several autonomous-operation rules,
including:

-   continue autonomously when a task requires multiple tool calls;
-   continue through discovered readable files for read-only batch
    tasks;
-   do not ask whether to continue when known batch work remains;
-   when the full contents of a truncated file are required, continue
    reading with offsets until the complete file has been read;
-   after a read result, do not answer unless the user requested an
    answer.

These rules are important when interpreting the subsequent model
behaviour.

------------------------------------------------------------------------

## 4. Model Behaviour: List → Read Transition

The trace establishes the transition precisely.

  ----------------------------------------------------------------------------------------------------
                Trace sequence UTC timestamp         Model-visible action
  ---------------------------- --------------------- -------------------------------------------------
                             1 15:58:42              User asks: `List the files in this directory`

                             3 16:01:15              Previous `bash {"command":"ls -1"}` call is
                                                     present

                             5 16:01:31              Qwen has now additionally called
                                                     `read {"path":"logfile_120K.log","limit":2000}`

                             7 16:15:27              Additional read from offset 33

                             9 16:40:48              Additional read from offset 65

                            11 17:13:13              Additional read from offset 172

                            13 18:25:42              Additional read from offset 283

                            15 20:38:37              Additional read from offset 398

                            17 21:15:05              Additional read from offset 509

                            19 22:37:40              Additional read from offset 620
  ----------------------------------------------------------------------------------------------------

The significant behavioural transition therefore occurred **immediately
after the successful `ls -1` result and before trace sequence 5**.

There was no additional user instruction directing Qwen to read the
file.

### Interpretation

The evidence supports the conclusion that the continuation originated in
model interpretation of the effective system prompt rather than from the
user.

A plausible mechanism is that Qwen interpreted the discovered file as a
read-only batch item and combined that interpretation with the prompt's
autonomous-continuation and full-file-reading rules.

This is a behavioural interpretation, not proof of the model's hidden
reasoning. What is directly evidenced is:

1.  the user requested only a directory listing;
2.  `ls -1` satisfied that request;
3.  no further user instruction was supplied;
4.  Qwen nevertheless initiated `read`;
5.  the effective system prompt contained rules capable of encouraging
    autonomous continuation through discovered readable material.

This distinction should be preserved in future analysis.

------------------------------------------------------------------------

## 5. Rolling Compaction --- Generation 1

At **12:49:00 local time**, Moderari created and persisted rolling
checkpoint generation 1.

### Compaction metrics

  Metric                                     Generation 1
  ------------------------------------ ------------------
  Configured trigger                               63.00%
  Actual trigger                                   71.50%
  Estimated context after compaction               34.09%
  Continuity size                        1,065 characters
  Produced distillation tokens                        557
  Covered non-system messages                           9
  Newly covered messages                                8
  Removed messages                                      8
  Recent chunks retained                                2
  Newly checkpointed source range            lines 0--191
  Retained recent source range             lines 172--417
  Persistence                                  Successful

The trigger being configured at 63% does **not** mean compaction
necessarily occurs at exactly 63%. In this execution the next applicable
compaction opportunity occurred at an estimated 71.50%.

### Continuity quality

Generation 1 retained the original objective accurately:

> List the files in this directory.

It also represented the task state as:

> The task is completed.

and:

> Next required action: None

It recorded the established fact that the directory contained only
`logfile_120K.log`.

The checkpoint therefore did **not** semantically transform the user's
request into a request to read the file.

------------------------------------------------------------------------

## 6. Behaviour After Generation 1

This is one of the strongest observations from the experiment.

After generation 1, the authoritative checkpoint supplied back to the
model explicitly stated:

-   the objective was to list the files;
-   the task was completed;
-   the directory contained one file;
-   current working strategy was `None`;
-   next required action was `None`.

Despite that state, Qwen continued issuing reads against
`logfile_120K.log`.

For example, the trace subsequently contains reads at offsets 398 and
later offsets.

This means the continued file reading cannot reasonably be attributed to
semantic drift in the generation-1 distilled continuity. The checkpoint
correctly preserved the original objective and completion state.

The continued behaviour instead remained associated with the model's
execution behaviour under the wider system prompt and surviving/recent
conversational context.

------------------------------------------------------------------------

## 7. Rolling Compaction --- Generation 2

At **17:21:40 local time**, the same uninterrupted session underwent a
second rolling compaction.

### Compaction metrics

  Metric                                     Generation 2
  ------------------------------------ ------------------
  Configured trigger                               63.00%
  Actual trigger                                   69.07%
  Estimated context after compaction               30.96%
  Continuity size                        1,065 characters
  Produced distillation tokens                        492
  Covered non-system messages                          15
  Newly covered messages                                6
  Removed messages                                      6
  Recent chunks retained                                2
  Newly checkpointed source range          lines 172--528
  Retained recent source range             lines 509--659
  Persistence                                  Successful
  Previous checkpoint                        Generation 1

Generation 2 was explicitly linked to generation 1 through
`previous_checkpoint_id`.

The second compaction therefore demonstrates checkpoint succession
rather than two unrelated compaction events.

### Semantic continuity

Generation 2 again retained:

-   the original user objective;
-   the correct completion condition;
-   the fact that the task was already complete;
-   the fact that the directory contained only `logfile_120K.log`;
-   no secondary responsibility;
-   no next required action.

There is no observed semantic drift in the controlling objective between
generations 1 and 2.

------------------------------------------------------------------------

## 8. Context Behaviour Across Both Compactions

The central compaction sequence is:

``` text
Initial execution
      ↓
Context growth
      ↓
71.50%
      ↓
Generation 1
      ↓
34.09%
      ↓
Continued uninterrupted execution
      ↓
Context growth
      ↓
69.07%
      ↓
Generation 2
      ↓
30.96%
      ↓
Continued uninterrupted execution
      ↓
Terminal model answer
```

This validates the principal rolling-compaction requirement more
strongly than a single checkpoint would.

Moderari successfully compacted an already-compacted session and
continued operating from the resulting continuity state.

------------------------------------------------------------------------

## 9. Terminal Behaviour

At **18:01:50 local time**, the model returned:

> `The task is completed. The directory contains only one file: logfile_120K.log.`

Moderari recorded:

-   `finish_reason=stop`;
-   answer length: 78 characters;
-   a persisted terminal checkpoint;
-   terminal generation: 3;
-   previous checkpoint: generation 2.

The terminal answer is semantically consistent with the original user
instruction and with both rolling checkpoints.

### Important qualification: the file was not read to EOF

The final Moderari `completion_report` records:

``` text
eof_status=incomplete
outstanding_read=('logfile_120K.log', 620)
```

This changes one assumption made while observing the run interactively.

The execution **did finish**, but Qwen did **not** finish reading the
entire 120K file before answering.

The last trace request contains a read at offset 620, while Moderari's
terminal state still considers that source read outstanding/incomplete.

Therefore the accurate conclusion is:

> Qwen autonomously read substantial portions of a file that the user
> had never asked it to read, survived two rolling context compactions,
> and then eventually abandoned that unnecessary continuation and
> returned to the already-completed original objective.

That behaviour is arguably more interesting than simply reading to EOF.

------------------------------------------------------------------------

## 10. What the Test Establishes

### Moderari / Lumen

**PASS --- repeated rolling compaction**

Two successive rolling context compactions occurred within one
uninterrupted session.

**PASS --- context reduction**

Generation 1 reduced estimated utilisation from 71.50% to 34.09%.

Generation 2 reduced it from 69.07% to 30.96%.

**PASS --- checkpoint persistence**

Both rolling checkpoints were persisted successfully.

A terminal generation-3 checkpoint was also persisted.

**PASS --- checkpoint lineage**

Generation 2 references generation 1, and the terminal checkpoint
references generation 2.

**PASS --- objective preservation**

The original instruction remained:

> `List the files in this directory`

through both rolling checkpoints and the terminal answer.

**PASS --- completion-state preservation**

Both rolling checkpoints correctly recognised that the explicit user
task had already been completed.

**PASS --- execution survived repeated compaction**

The same model session continued after generation 1 and again after
generation 2.

**PASS --- source coverage tracking**

Moderari retained and evolved source-range evidence across the
checkpoint generations.

**PASS --- terminal semantic recovery**

The final model answer returned to the original user objective without
visible semantic drift.

------------------------------------------------------------------------

## 11. Behavioural Finding

The test exposes a separate model/orchestration behaviour that should
not be confused with a Moderari compaction defect.

Qwen continued beyond the user's completed request and began reading the
only discovered file.

The trace demonstrates that this behaviour started **before the first
compaction**.

Generation 1 subsequently stated explicitly that the task was complete
and that no next action was required, yet Qwen continued reading.

Generation 2 preserved the same state.

The behaviour therefore:

-   was not caused by generation-1 compaction;
-   was not caused by generation-2 compaction;
-   was not caused by loss of the original objective;
-   was not caused by the checkpoint incorrectly instructing the model
    to continue.

The most credible evidenced explanation is an interaction between Qwen
and Moderari's default system-prompt rules governing autonomous work,
read-only batches and truncated/full-file reads.

This warrants a controlled follow-up experiment, but it does **not**
require Moderari to suppress the behaviour. For Lumen's research
purpose, allowing and recording such continuation is desirable.

------------------------------------------------------------------------

### Compaction as a possible behavioural influence

A further observation arises from the timing of the model's eventual return to the original objective.

**Compaction may have influenced subsequent model behaviour by repeatedly reasserting the distilled task state and reducing the accumulated contextual momentum of the model's unnecessary continuation.**

This is a hypothesis supported by temporal correlation, not a demonstrated causal conclusion. Generation 1 and Generation 2 both reasserted that the original task was complete, that there was no secondary responsibility, and that no next action was required. After Generation 2, Qwen requested another source chunk but subsequently stopped the unnecessary reading before EOF and returned to the original completed objective.

One possible interpretation is that repeated compaction altered the relative salience of the model-visible context: accumulated material associated with the self-directed file-reading activity was reduced, while the authoritative distilled continuity repeatedly restored the original objective and completed-task state.

This reinforces the M0.1 requirement that context compaction be treated as part of the experimental condition rather than assumed to be behaviourally neutral. A controlled follow-up experiment is required before attributing causality.

---

## 12. Pi vs Moderari Context Percentages

During the execution Pi displayed context figures substantially
different from Moderari's figures.

These values should not currently be treated as contradictory.

The evidence from this test establishes Moderari's own measured
compaction transitions, but does not establish that Pi and Moderari
calculate context utilisation from the same message set, token
estimator, denominator, or lifecycle point.

This discrepancy should be investigated separately and must not be used
to invalidate the compaction result without first establishing the
semantics of both measurements.

------------------------------------------------------------------------

## 13. Timing and Resource Behaviour

The run took many hours and individual model/distillation phases were
unusually long.

The test environment was also observed under substantial memory
pressure, including high llama-server memory consumption which was later
released.

No performance conclusion should be drawn from this run.

Timing was not the purpose of the experiment, and the evidence does not
isolate:

-   model inference performance;
-   Windows memory pressure;
-   paging;
-   llama-server behaviour;
-   other concurrently running Lumen services;
-   accumulated development-environment state; or
-   other machine-level resource contention.

Performance should therefore remain explicitly **out of scope** for the
validation result.

------------------------------------------------------------------------

## 14. Overall Assessment

### Result: PASS, with behavioural evidence requiring follow-up

The M0.1 Moderari rolling context-compaction mechanism successfully
demonstrated:

1.  threshold-triggered compaction;
2.  substantial context reduction;
3.  persisted distilled continuity;
4.  reinjection of authoritative continuity;
5.  uninterrupted execution after compaction;
6.  a second successful compaction of the same session;
7.  checkpoint lineage;
8.  preservation of the original objective and completion state;
9.  source-range continuity; and
10. normal terminal completion with a terminal checkpoint.

No evidence in this run indicates semantic drift caused by compaction.

The unexpected autonomous reading behaviour is real and valuable, but
the trace places its origin **before compaction** and provides evidence
that the effective default system prompt may have contributed to it.

The model eventually stopped the unnecessary continuation before EOF and
returned the correct answer to the original request.

------------------------------------------------------------------------

## 15. Recommended Follow-Up Experiment

The next experiment should isolate the **list → read** transition rather
than repeat the expensive 120K compaction run immediately.

Use a small directory containing one small readable file and repeat:

``` text
List the files in this directory
```

under controlled prompt conditions.

Suggested comparison:

  -----------------------------------------------------------------------
  Run                     Moderari policy /       Expected observation
                          prompt condition        
  ----------------------- ----------------------- -----------------------
  A                       Current Moderari        Does Qwen list and then
                          Default                 read?

  B                       Pass-through using Pi's Does Qwen stop after
                          original system prompt  listing?

  C                       Moderari Default with   Does the behaviour
                          the read-only batch     change?
                          continuation wording    
                          removed/altered         
  -----------------------------------------------------------------------

The purpose is not to force a preferred behaviour. It is to determine
which orchestration condition changes the observed behaviour.

This would convert the accidental observation from the 120K run into a
controlled behavioural experiment suitable for later Aestimare analysis.

------------------------------------------------------------------------

## 16. Final Evidence Statement

**The 29 August 2026 120K-file execution provides successful evidence of
repeated rolling context compaction in a single uninterrupted Moderari
session. Two rolling generations reduced context from 71.50% → 34.09%
and 69.07% → 30.96%, respectively, while preserving the original
objective and completed-task state. A terminal generation-3 checkpoint
was subsequently persisted. The model's unexpected decision to read the
discovered file began before compaction, continued despite checkpoints
explicitly recording that no further action was required, and ended
before file EOF when the model returned to the original completed
objective.**
