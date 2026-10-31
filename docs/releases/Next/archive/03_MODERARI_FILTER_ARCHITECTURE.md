# Lumen Moderari Filter Architecture

**Status:** Current authoritative architecture  
**Date:** 02026-10-10  
**Scope:** Filter contract and the first production System Prompt Filter

## 1. Purpose

Filters are Moderari's explicit mechanism for intentional interaction transformation or observation.

The central distinction is:

```text
Adapter
    → representation may change
    → semantics should not

Filter
    → semantics may deliberately change
```

All Filters operate on CIM and are therefore independent of external protocol representation.

## 2. Filter Boundary

A transformational Filter has the form:

```text
CIM₁ → Filter → CIM₂
```

An observational/private-model Filter may perform one or more private model interactions while retaining a single Filter execution identity:

```text
CIM
 ↓
Filter
 ↓
FilterAsk → SIE/Core → provider/model
 ↓
FilterResult
 ↓
same Filter execution
```

Private model interaction does not create a persistent conversation branch.

## 3. Ownership

Filters do not own:

- conversation persistence;
- session persistence;
- provider transport;
- model catalogue knowledge;
- protocol translation;
- user configuration persistence.

Moderari owns/persists user configuration. At session materialisation, the selected Filter configuration becomes immutable execution configuration supplied to the Filter.

## 4. Selection

Filters are explicitly selected by the user/session configuration, normally through a Filter Profile. `None` / no Filter Profile remains a valid choice where allowed.

SIE must never search for or automatically select a Filter to compensate for Adapter OUT incompatibility.

## 5. Filter Profiles

A Filter Profile is a named collection of Filter identities and configuration. Profiles are configuration conveniences, not replay authorities.

Profiles are not versioned as independent historical execution artefacts. Replay/evidence records the exact Filters, versions and effective configuration used by the interaction.

## 6. Filter Manifest and Discovery

Filters are self-describing/discoverable components. The production discovery contract will build on the real BaseFilter and SystemPromptFilter rather than on test doubles.

A Filter declares identity/version, active CIM version compatibility, execution mode, and any required EMD characteristic keys.

## 7. Effective Model Definition

A Filter may declare `requires` containing EMD characteristic keys. `requires: []` is valid.

SIE checks existence of required keys. The Filter owns interpretation of their values. EMD is not passed merely so a Filter can become a second model catalogue.

## 8. System Prompt Filter

The first production Filter is intentionally simple.

Its configured semantic operation is:

> Insert the user-configured system prompt as the initial interaction instruction, or replace the existing initial system instruction if one already occupies that initial position.

It does **not** replace every system-role/system-authority instruction in the conversation.

### 8.1 Replacement

Input:

```text
0 system      Original initial instruction
1 user        Hello
2 assistant   Hello
```

Output:

```text
0 system      Configured system prompt
1 user        Hello
2 assistant   Hello
```

### 8.2 Insertion

If the interaction does not begin with a system instruction:

```text
0 user        Hello
1 assistant   Hello
```

the Filter inserts:

```text
0 system      Configured system prompt
1 user        Hello
2 assistant   Hello
```

with the resulting CIM chronology remaining valid.

### 8.3 Later system instructions

Input:

```text
0 system      Original initial instruction
1 user        Hello
2 assistant   Hello
3 system      Later behavioural instruction
4 user        Continue
```

Output:

```text
0 system      Configured system prompt
1 user        Hello
2 assistant   Hello
3 system      Later behavioural instruction
4 user        Continue
```

The later system instruction is untouched.

### 8.4 Initial developer instruction

An initial developer instruction is not silently reinterpreted as the system instruction being replaced. The Filter inserts/replaces according to the explicit initial-system rule while preserving other interaction semantics.

## 9. Configuration Persistence

The Filter itself has no MongoDB or persistence awareness.

```text
User
  ↓
Moderari persisted prompt
  ↓
Session Execution Plan / immutable configuration
  ↓
SystemPromptFilter
  ↓
CIM
```

This keeps persistence outside the extension implementation.

## 10. Validation

A production Filter accepts valid CIM and must return valid CIM. The active CIM schema is therefore a hard component boundary.

A Filter cannot invent private CIM fields. If a new semantic is required, that is a CIM evolution decision.

## 11. Transformation Evidence

The System Prompt Filter produces deterministic transformation evidence sufficient to distinguish at least:

- `insert_initial_system_prompt`;
- `replace_initial_system_prompt`;
- affected CIM location;
- Filter identity/version.

Moderari owns persistence of Filter transformation evidence.

## 12. No Default Moderari System Prompt

Vanilla Moderari does not impose a hidden/default system prompt. A custom system prompt is an intentional Filter transformation selected through configuration.

## 13. Cognition / Observational Filters

The Cognition Filter remains a future observational/private-model Filter example. It is useful architecturally because it exercises a different Filter execution mode, but it is not required to establish the first production discovery contract.

## 14. Production Status

Through Milestone 5E:

- `BaseFilter` production contract exists;
- `SystemPromptFilter` exists;
- CIM input/output validation is enforced;
- insertion/replacement behaviour is tested;
- later system instructions are preserved;
- transformation evidence is emitted;
- persistence remains outside the Filter.

## 15. Architectural Invariants

1. Filters consume and return CIM.
2. Filters are protocol-independent.
3. Filters are explicitly selected.
4. Intentional semantic change belongs in a Filter.
5. Filters do not repair protocol/provider representation oddities.
6. Filters do not own persistence or conversation context.
7. Filter output must conform to the active CIM schema.
8. System Prompt Filter affects only the initial system instruction.
9. Later system instructions remain untouched.
10. Exact Filter identity/version/configuration is execution provenance.
