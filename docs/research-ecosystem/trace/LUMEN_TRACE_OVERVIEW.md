# Lumen Trace

## Transparent Capture and Recording for the Lumen Research Ecosystem

---

# Purpose

Lumen Trace is the first project within the Lumen Research Ecosystem.

Its purpose is simple:

> Capture every interaction between Pi and Lumen exactly as it occurred, without modifying either side of the conversation.

Lumen Trace is **not** part of Lumen.

Lumen Trace is **not** part of Pi.

It is an independent application that sits transparently between the two systems and records their communication.

The recorded interactions become immutable datasets that can later be replayed and analysed by other projects within the ecosystem.

---

# Why Lumen Trace Exists

During development of Lumen we repeatedly observed an interesting phenomenon.

Using:

* the same AI model
* the same prompt
* the same files
* the same execution environment
* the same Lumen version

the quality of the final answer could vary significantly between executions.

The objective of the Lumen Research Ecosystem is to investigate this behaviour scientifically.

However, meaningful experiments require repeatable inputs.

Without capturing an interaction exactly as it occurred there is no way to determine whether differences between executions were caused by:

* different inputs
* different timing
* different orchestration
* different model behaviour
* environmental changes

Lumen Trace solves this problem by producing a permanent recording of a real interaction.

These recordings become the foundation for future replay experiments.

---

# Design Philosophy

Lumen Trace follows a strict design philosophy.

It performs one responsibility only.

**Observe.**

It does not:

* analyse
* replay
* modify
* optimise
* evaluate
* interpret

It records.

Nothing more.

Keeping this responsibility isolated ensures that every recording is an accurate representation of what actually occurred.

---

# High Level Architecture

```
                 Normal Operation

                Pi
                 │
                 │
                 ▼
        ┌─────────────────┐
        │   Lumen Trace   │
        │                 │
        │ Capture & Store │
        └─────────────────┘
                 │
                 ▼
               Lumen
                 │
                 ▼
           Ollama / Qwen
```

Pi communicates with Lumen Trace exactly as it previously communicated with Lumen.

Lumen is simply moved to another port.

Pi remains completely unaware that Trace exists.

---

# Transparency

One of the primary design goals is complete transparency.

Pi should behave exactly as if it were communicating directly with Lumen.

Lumen should behave exactly as if Pi were communicating directly with it.

The only difference is that every message is recorded.

The forwarding process must preserve:

* HTTP method
* URL
* query parameters
* request headers
* response headers
* request body
* response body
* HTTP status
* ordering

No modifications are made.

---

# Responsibilities

Lumen Trace is responsible for:

* accepting incoming Pi requests
* forwarding requests to Lumen
* forwarding responses back to Pi
* recording every interaction
* storing recordings in MongoDB
* presenting a live trace view
* allowing recordings to be named

Lumen Trace is **not** responsible for:

* replaying conversations
* evaluating outputs
* model management
* checkpoint analysis
* AI orchestration

---

# Recording Lifecycle

Every recording begins with a user supplied name.

Example:

```
operations.py baseline
```

When recording begins Trace automatically appends a timestamp.

Example:

```
operations.py baseline_20260801_084713
```

This becomes the permanent recording identifier.

A recording progresses through three states.

```
Idle
    │
    ▼
Recording
    │
    ▼
Completed

or

Recording
    │
    ▼
Aborted
```

Only completed recordings are available for replay.

---

# MongoDB

Lumen Trace stores data inside the existing Lumen database.

Initially two collections are required.

## trace_recordings

Stores recording metadata.

Example:

```json
{
    "_id": "...",
    "name": "operations.py baseline_20260801_084713",
    "started_at": ISODate("2026-08-01T08:47:13Z"),
    "completed_at": ISODate("2026-08-01T08:49:21Z"),
    "status": "completed"
}
```

---

## trace_messages

Stores every captured request and response.

Each message contains:

* recording id
* sequence number
* exchange id
* session id
* timestamp
* direction
* HTTP method
* URL path
* query string
* status code
* headers
* JSON message
* request duration

This schema is intentionally generic so it remains valid as Trace evolves.

---

# User Interface

The first version intentionally remains simple.

```
+------------------------------------------------------+

 Recording:
 [ operations.py baseline            ] [Start Trace]

--------------------------------------------------------

Status: Recording

Recording:
operations.py baseline_20260801_084713

Session:
6e4d....

--------------------------------------------------------

Seq    Time         Direction

1      08:47:13     Pi → Lumen

2      08:47:15     Lumen → Pi

3      08:47:15     Pi → Lumen

4      08:47:18     Lumen → Pi

--------------------------------------------------------

                 [ Stop Trace ]

+------------------------------------------------------+
```

Selecting a message displays formatted JSON.

---

# Configuration

Configuration is provided using a single YAML file.

```yaml
server:
  host: 127.0.0.1
  port: 11435

lumen:
  host: 127.0.0.1
  port: 11436

mongodb:
  uri: mongodb://...
  database: lumen

  recordings_collection: trace_recordings
  messages_collection: trace_messages

  server_selection_timeout_ms: 3000
  connect_timeout_ms: 3000
```

---

# Initial Experiment

The first recorded experiment will analyse:

```
operations.py
```

located at

```
C:\Development\EF-Social_Discovery-BOB\
src\ef_social_discovery\api\routes\
operations.py
```

This file is approximately 15 KB in size and therefore fits comfortably within the available context window without requiring compaction.

It has already demonstrated observable differences in answer quality between repeated executions, making it an ideal candidate for the first controlled experiment.

---

# Relationship to Other Projects

Lumen Trace is the first stage of the research pipeline.

```
Lumen Trace
      │
      ▼
Lumen Replay
      │
      ▼
Lumen Assessment
      │
      ▼
Lumen Servire
```

Trace produces immutable recordings.

Replay consumes those recordings.

Assessment analyses replay results.

Servire provides the operational dashboard across the complete ecosystem.

Each project has a single responsibility.

---

# Future Enhancements

Future versions may include:

* recording search
* recording tags
* recording export/import
* message filtering
* protocol statistics
* response timing visualisation
* multiple concurrent recordings
* session browsing
* REST API for external tooling

These enhancements must not compromise the primary design objective:

> Produce an accurate, transparent and reproducible recording of every interaction between Pi and Lumen.

---

# Success Criteria

The first release of Lumen Trace will be considered successful when:

* Pi communicates through Trace without modification.
* Lumen operates unchanged behind Trace.
* Every request and response is recorded.
* Messages are stored correctly in MongoDB.
* The user can create, start and stop recordings.
* Recordings are uniquely named.
* The UI displays the interaction in real time.
* Completed recordings are available for future replay.

At that point Lumen Trace becomes the trusted source of experimental data for the remainder of the Lumen Research Ecosystem.
