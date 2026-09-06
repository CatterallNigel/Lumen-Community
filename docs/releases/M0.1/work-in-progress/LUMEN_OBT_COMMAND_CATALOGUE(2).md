# Lumen `\obt` User Command Catalogue

**Status:** Living canonical user-command register  
**Initial catalogue date:** 2026-08-27  
**Last reconciled:** 2026-09-06  
**Scope:** Commands that a researcher or authorised operator may enter through a
Lumen client such as Pi or Rogare

## Document Revision History

| Date | By | Version | Description |
| --- | --- | --- | --- |
| 2026-08-27 | Nigel Catterall / OpenAI | 0.1 | Initial mixed user/internal catalogue |
| 2026-09-06 | Nigel Catterall / OpenAI | 0.2 | Corrected the prefix and service-qualified syntax; retained only user-entered commands |

## 1. Purpose and Boundary

This document records the Lumen control commands that a researcher or authorised
operator may type into a connected client.

The canonical prefix is `\obt`, not `/obt`. Commands are service-qualified:

```text
\obt <service> <operation> [arguments...]
```

These commands are intercepted by Lumen and must not be sent to the model or added
to model context or Vestigare conversational Trace evidence.

This catalogue deliberately excludes:

- all service-to-service bootstrap, lifecycle, routing, notification, execution,
  recording-control and provenance commands;
- proposed or reserved commands that do not have a current user contract; and
- service lifecycle actions performed through the Servire UI.

## 2. Command Index

| Command | Owner | Audience | Purpose |
| --- | --- | --- | --- |
| `\obt pontis session status` | Pontis | Researcher | Show the current client's authoritative session state |
| `\obt pontis session tools status` | Pontis | Researcher | Show whether provider-tool injection is enabled |
| `\obt pontis session tools on` | Pontis | Researcher | Enable provider-tool injection for the current session |
| `\obt pontis session tools off` | Pontis | Researcher | Disable provider-tool injection for the current session |
| `\obt pontis session end` | Pontis | Researcher | End the current session gracefully |
| `\obt pontis session new` | Pontis | Researcher | End the current session and create a replacement |
| `\obt pontis sessions` | Pontis | Authorised operator | List authoritative sessions |
| `\obt pontis session end <session_id>` | Pontis | Authorised operator | Gracefully end a named session |
| `\obt pontis session force-close <session_id>` | Pontis | Authorised operator | Force-close an orphaned or unrecoverable session |
| `\obt praebere help` | Praebere | Researcher | List Praebere commands |
| `\obt praebere providers` | Praebere | Researcher | Show authoritative provider state |
| `\obt praebere models` | Praebere | Researcher | Show the discovered model catalogue and selection state |
| `\obt praebere model select <model_name>` | Praebere | Researcher | Reserve the selected runtime-global model for the current session |
| `\obt praebere reconcile` | Praebere | Authorised operator | Retry reconciliation of persisted runtime state with Pontis |
| `\obt moderari help` | Moderari | Researcher | List Moderari commands |
| `\obt moderari status` | Moderari | Researcher | Show current Moderari session/model/profile status |
| `\obt moderari context` | Moderari | Researcher | Show estimated context usage |
| `\obt moderari prompt status` | Moderari | Researcher | Show the current session's system-prompt policy |
| `\obt moderari prompt default` | Moderari | Researcher | Apply the default system-prompt policy to the current session |
| `\obt moderari prompt pass-through` | Moderari | Researcher | Apply Pass-through to the current session |
| `\obt moderari session info` | Moderari | Researcher | Show the current Moderari session identity and state |
| `\obt moderari session list` | Moderari | Researcher | List saved Moderari sessions |
| `\obt moderari session resume <number_or_session_id>` | Moderari | Researcher | Stage a saved Moderari session for restoration |
| `\obt moderari session continuation` | Moderari | Researcher | Show continuation state for the current session |
| `\obt repetere help` | Repetere | Researcher | Show Repetere command help |
| `\obt repetere list` | Repetere | Researcher | List prepared Replay sessions |
| `\obt repetere start <replay_id>` | Repetere | Researcher | Start the identified prepared Replay |
| `\obt repetere status` | Repetere | Researcher | Show the applicable Replay status |
| `\obt repetere stop` | Repetere | Researcher | Stop the applicable active Replay |

## 3. Pontis Session Commands

### Client-scoped commands

The following commands apply only to the session from which they are sent:

```text
\obt pontis session status
\obt pontis session tools status
\obt pontis session tools on
\obt pontis session tools off
\obt pontis session end
\obt pontis session new
```

`session tools on` and `session tools off` change provider-tool injection without
changing the model reserved for the session. `session end` is idempotent.
`session new` ends the current session before returning a replacement session
identity.

### Restricted operator commands

```text
\obt pontis sessions
\obt pontis session end <session_id>
\obt pontis session force-close <session_id>
```

These commands can name another session and therefore require the authorised
operator path. Normal end is preferred. Force-close is a conspicuous recovery action
for an orphaned session or failed graceful closure.

## 4. Praebere Model Commands

```text
\obt praebere help
\obt praebere providers
\obt praebere models
\obt praebere model select <model_name>
\obt praebere reconcile
```

`providers` and `models` report Praebere's authoritative cached state. Ordinary
queries do not refresh Ollama discovery; discovery occurs at Praebere startup and
through the Praebere UI's **Refresh Models** action.

`model select` attaches the current session to the named runtime-global model. The
model must already be in Praebere's discovered catalogue. Selection is idempotent for
the same session and model. A conflicting selection is rejected while another open
session reserves the current model or while model execution is active.

`reconcile` is an operator recovery command. It retries reconciliation of the
persisted `praebere_runtime_state` with Pontis and returns the authoritative outcome.

## 5. Moderari Session and Prompt Commands

```text
\obt moderari help
\obt moderari status
\obt moderari context
\obt moderari prompt status
\obt moderari prompt default
\obt moderari prompt pass-through
\obt moderari session info
\obt moderari session list
\obt moderari session resume <number_or_session_id>
\obt moderari session continuation
```

Prompt-policy changes are session-scoped. `prompt status` reports only the active
mode. Custom prompt content is applied through the Moderari UI in M0.1.

`session resume` stages the chosen saved session. Its saved context is restored on
the next ordinary, non-command message; the command itself does not invoke the model.

## 6. Repetere Replay Commands

```text
\obt repetere help
\obt repetere list
\obt repetere start <replay_id>
\obt repetere status
\obt repetere stop
```

`list` returns prepared Replay sessions. `start` starts the identified prepared
Replay in a fresh isolated Replay session. `status` and `stop` apply to the current
Replay context.

Unstage and delete operations are not included because no current user `\obt`
contract has been established for them.

## 7. Common Outcomes

| Outcome | Meaning |
| --- | --- |
| `200 OK` | The owner completed the command; the body contains the authoritative result where applicable |
| `204 No Content` | The addressed service did not handle the command |
| `4xx` | The command was invalid, unauthorised or could not be applied to the current state |
| `5xx` | The owning service or a required dependency failed while executing the command |
| `504 Gateway Timeout` | Lumen could not confirm the authoritative outcome before the configured timeout |

Every solicited user command must receive one terminal response. Silence is not
success. A late owner response after a `504` remains diagnostic evidence and does not
replace the terminal response already returned to the client.

## 8. Maintenance Rule

Add a command to this catalogue only when it is deliberately available for direct
researcher or authorised-operator entry. Internal commands belong in service
contracts and implementation documentation, not in this user catalogue.
