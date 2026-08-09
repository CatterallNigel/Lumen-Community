# Servire Roadmap Addendum
**Date:** 2026-08-08

## Overview

During integration of Lumen Replay with automatic Trace recording, several operational improvements were identified for Servire. These do not alter the core orchestration responsibilities of Servire but significantly improve day-to-day operation, debugging, testing and support.

---

# 1. Operational Log Export

## Motivation

The Servire Operational Log is rapidly becoming the primary view of the Lumen++ stack.

During investigation of Replay/Trace behaviour it became clear that collecting individual logs from:

- Servire
- Lumen
- Replay
- Trace

is unnecessarily cumbersome.

Providing an export facility allows a complete operational timeline to be preserved alongside bug reports and engineering investigations.

## Requirements

Add an **Export** button to the Operational Log page.

The exported file should contain:

- timestamps
- component
- severity
- message
- structured context (where available)

Suggested filename:

```
servire-operational-log_YYYYMMDD_HHMMSS.log
```

or

```
servire-operational-log_YYYYMMDD_HHMMSS.jsonl
```

Future enhancement:

- Export only current session
- Export filtered log entries
- Export complete history

---

# 2. Component Log Management

## Motivation

Lumen already supports clearing its operational log on startup.

Replay and Trace currently require manual deletion of log files before testing.

Servire should become the single operational console responsible for managing logs across the Lumen++ stack.

Rather than manipulating log files directly, Servire should invoke control APIs exposed by each component.

This preserves clear ownership boundaries:

- each component owns its own logs
- Servire orchestrates operations

---

## Proposed API

Each managed component should expose a control endpoint similar to:

```
POST /logs/clear
```

Future endpoints may include:

```
POST /logs/archive
GET  /logs/download
```

---

## Initial Servire Behaviour

When requested by the operator:

```
Clear All Logs
```

Servire performs:

```
POST Lumen  /logs/clear
POST Replay /logs/clear
POST Trace  /logs/clear
```

Each component is responsible for:

- flushing log writers
- safely closing files
- removing or recreating the active log
- reporting success/failure

Servire records each operation within its Operational Log.

Example:

```
14:32:18  Clearing component logs...

14:32:18  Lumen  ........ OK
14:32:18  Replay ........ OK
14:32:19  Trace  ........ OK

14:32:19  Log clear completed
```

---

# Future Enhancements

Potential future capabilities include:

- Archive all component logs
- Download all component logs
- Bundle all logs into a single ZIP
- Automatic log archival before stack startup
- Retention policies
- Scheduled cleanup

---

# Architectural Notes

These features continue the existing Servire design philosophy:

- Servire orchestrates.
- Individual components own their own operational state.
- Cross-component interaction occurs exclusively through public control APIs.
- Servire becomes the operational console for the Lumen++ ecosystem rather than directly manipulating component internals.

This approach maintains clear separation of responsibilities while providing operators with a significantly improved debugging and support experience.