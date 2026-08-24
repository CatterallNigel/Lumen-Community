# Servire User Manual — Overview and Outline

**Release:** Lumen M0.1  
**Status:** Planned  
**Document purpose:** Define the intended structure and approach for the Servire User Manual.

---

## 1. Purpose

The M0.1 external/research distribution should include a user-oriented manual for **Lumen Servire**.

The manual is intended to explain how to operate the Lumen runtime through the Servire user interface.

It is not intended to duplicate Lumen's architecture, implementation, or developer documentation.

The distinction is:

> **Technical documentation explains how Lumen works.  
> The Servire User Manual explains what the user sees and what they can do.**

The manual should therefore be primarily visual and task-oriented.

---

## 2. Documentation Approach

The manual should be based predominantly around **annotated screenshots of the Servire UI**.

Each significant action, control, indicator, or status area shown in a screenshot should be identified using a numbered callout.

For example:

**① Service Status**  
Shows the current operational state of the service.

**② Start**  
Starts the selected service after the required Servire validation has completed.

**③ Stop**  
Stops the service.

**④ Restart**  
Performs a controlled restart of the service.

The corresponding numbers should be visibly overlaid on the screenshot.

Descriptions should remain deliberately short. The screenshot provides the context; the accompanying text explains what the identified item means or does.

---

## 3. Consistent Visual Language

The same annotation conventions should be used throughout the manual.

Where practical, common Servire controls should retain the same callout numbers between chapters.

For example, if **③** represents the Stop action in one service chapter, the same number should represent Stop in other service chapters where the UI structure permits it.

Service-specific controls or information can then receive additional callout numbers.

This should allow a reader who has understood one Servire service screen to recognise the common operational concepts immediately when viewing another.

---

## 4. Chapter Structure

Each Lumen service represented within Servire should have its own chapter.

A typical service chapter should contain:

### 4.1 Service Name and Role

The Lumen service name and its user-facing role.

For example:

> **Lumen Moderari — Orchestration Service**

### 4.2 What This Service Does

A very short explanation of the purpose of the service.

This should describe the service from an operator's perspective rather than explain its internal architecture.

### 4.3 Annotated Servire Screenshot

A screenshot showing how the service is represented within Servire.

Controls, states, indicators, and other significant UI elements should be numbered.

### 4.4 Numbered Control Descriptions

A corresponding explanation for each numbered item.

Descriptions should normally be limited to one or two short paragraphs.

### 4.5 Operational Notes

Where necessary, the chapter may contain short notes explaining important user-visible behaviour, dependencies, restrictions, or expected transitions.

For example:

> Starting Moderari may first require Servire to verify that its required dependencies are available.

These notes should describe observable behaviour rather than implementation details.

---

## 5. Proposed Manual Structure

The exact structure should follow the final M0.1 Servire UI, but is expected to include approximately the following sections.

### 5.1 Introduction

Briefly explain:

- what Servire is;
- what the Servire UI represents;
- the concept of the Lumen service stack;
- service states;
- the difference between individual service controls and stack-level controls.

### 5.2 Servire / Stack Overview

Provide an annotated screenshot of the complete Servire interface.

Explain the major areas of the UI, including:

- overall stack state;
- global stack controls;
- service list;
- individual service state;
- health information;
- operational logging;
- other global indicators or controls.

### 5.3 Individual Service Chapters

Create a chapter for each service exposed through the M0.1 Servire UI.

Each chapter should use the common structure defined above.

The final list should be derived from the actual M0.1 distribution rather than being permanently fixed by this outline.

Likely chapters include services such as:

- Praebere;
- Moderari;
- Pontis;
- Vestigare / Trace;
- Repetere / Replay;
- Aestimare;
- Rogare;
- any additional runtime service exposed through Servire in M0.1.

### 5.4 Starting the Lumen Stack

Provide a visual walkthrough of starting the complete Lumen runtime.

The section should show the expected user-visible progression through states such as:

> Stopped → Starting → Running

Where Servire performs dependency validation or waits for services to become healthy, this should be explained from the user's perspective.

### 5.5 Stopping the Lumen Stack

Explain the expected stack shutdown procedure and the visible state changes within Servire.

Where shutdown causes managed resources to be released, this should be mentioned where it is relevant to the operator.

### 5.6 Service States

Explain the service states that a user may encounter in the M0.1 UI.

For example:

- Stopped
- Starting
- Running
- Stopping
- Unhealthy
- Failed

The final list should reflect the actual states exposed by Servire.

### 5.7 Operational Logs

Explain how to:

- locate the log for a service;
- interpret the basic log presentation;
- distinguish normal lifecycle information from errors;
- clear the displayed log where supported.

The manual should not attempt to document every possible log message.

### 5.8 Failures and Basic Troubleshooting

Provide guidance for common user-visible failure conditions.

This should focus on questions such as:

- What does a failed service look like?
- What happens when a dependency is unavailable?
- What should the user try first?
- When should a service be restarted?
- When should the complete stack be restarted?
- What information should be retained when reporting a problem?

This should remain basic operational guidance rather than becoming a developer troubleshooting manual.

---

## 6. Screenshot Requirements

Screenshots used in the manual should represent the **released M0.1 UI**, not intermediate development versions.

Screenshots should therefore be captured relatively late in the M0.1 release preparation process, once the Servire UI and expected operational behaviour have stabilised.

Each screenshot should:

- use a clean and representative Lumen installation;
- avoid irrelevant development information;
- show realistic service states;
- use clearly visible numbered annotations;
- remain readable at normal documentation size;
- use consistent annotation positioning and styling.

Original unannotated screenshots should also be retained so that annotations can be regenerated if the documentation changes.

---

## 7. Relationship to M0.1 Validation

Creating the Servire User Manual should also form part of the M0.1 usability review.

Documenting a control requires us to define what the user should expect when that control is used.

For example:

> Select **Start** → service enters **Starting** → validation succeeds → service enters **Running**.

If the actual behaviour cannot be described clearly, requires knowledge of Lumen internals, or produces confusing intermediate behaviour, that should be considered a potential usability issue before M0.1 is released.

The manual therefore serves two purposes:

1. **User documentation for the external/research distribution.**
2. **A practical review of the usability and predictability of Servire itself.**

---

## 8. Documentation Boundary

The Servire User Manual should deliberately avoid becoming another architecture document.

Where deeper information is required, the manual should reference the appropriate Lumen technical documentation rather than reproduce it.

As a general rule:

> **The manual documents the operator's interaction with Lumen.  
> The architecture documentation documents Lumen's internal design.**

---

## 9. Repository Organisation

The manual should live with the M0.1 release documentation.

A suitable structure would be:

```text
release/
└── M0.1/
    ├── SERVIRE_USER_MANUAL.md
    └── graphics/
        └── servire/
            ├── overview/
            ├── services/
            ├── workflows/
            └── troubleshooting/
```

This document is an outline only.

The final `SERVIRE_USER_MANUAL.md` and its screenshots should be created as part of the M0.1 release preparation once the Servire UI has reached sufficient stability.

---

## 10. M0.1 Objective

For M0.1, the Servire User Manual should be considered successful if an external researcher or evaluator can:

- understand the overall state of the Lumen runtime;
- identify the purpose of each service represented in Servire;
- start and stop the Lumen stack;
- start, stop, and restart individual services where permitted;
- understand normal service state transitions;
- locate and inspect operational logs;
- recognise an obvious failure condition;
- perform basic recovery actions;
- obtain useful information for reporting a problem;

without needing detailed knowledge of Lumen's internal architecture.

That should remain the guiding principle for the M0.1 Servire User Manual.