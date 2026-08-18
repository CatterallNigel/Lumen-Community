# Lumen v3.2.14.3 – Ask-Level Execution Aggregation

## Overview

This release corrects the Operational Intelligence UI so one user ask remains one execution across all Pi/model/tool round trips.

## Changes

- Intermediate tool-call responses no longer mark an execution complete.
- Execution start time, elapsed time, and timing totals persist across every model request in the same session.
- Tool calls and matching tool results accumulate into one ordered history.
- Successful reads populate session-wide source coverage.
- Added an **Execution Overview — Ask to Answer** panel showing the original ask, start time, live running duration, answer time, finish time, final status, and final answer.
- Final-answer receipt is recorded separately from persistence completion.
- Timeline, last activity, solution path, and terminal cognition now remain attached to the complete execution.

## Validation

Regression tests cover multi-round tool execution, ask preservation, answer capture, and terminal completion.
