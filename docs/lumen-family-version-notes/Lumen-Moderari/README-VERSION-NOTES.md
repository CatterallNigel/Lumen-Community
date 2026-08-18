# Version Notes

## Purpose

The documents in this directory provide a historical record of Lumen's development between version milestones.

Each document describes the significant engineering work completed for a specific development version, including architectural changes, implementation progress, behavioural improvements, and the reasoning behind major design decisions.

These documents are intended to explain **what changed** and **why**, rather than provide a complete description of the project at that point in time.

---

## Scope

Version Notes record development progress between versions.

They typically include:

* Objectives for the version.
* Significant architectural changes.
* New capabilities and features.
* Behavioural or operational improvements.
* Design decisions and their rationale.
* Areas deferred to future versions.
* Outstanding limitations or known issues where appropriate.

They are not intended to document every source code change or commit.

---

## Relationship to Other Documentation

The documentation within this repository serves different purposes.

| Location         | Purpose                                                                         |
| ---------------- | ------------------------------------------------------------------------------- |
| `README.md`      | Introduction to Lumen and the repository.                                       |
| `docs/`          | Current architecture, research, design principles, and reference documentation. |
| `version-notes/` | Historical record of changes between development versions.                      |

The documentation in `docs/` reflects the current state of the project.

The Version Notes explain how the project evolved to reach that state.

---

## Philosophy

Lumen places a strong emphasis on continuity, provenance, and engineering transparency.

Maintaining Version Notes provides a documented record of architectural evolution, allowing contributors to understand not only the current implementation, but also the reasoning behind significant changes over time.

The intention is to preserve engineering knowledge rather than simply record software releases.

---

## Reading the Version Notes

Individual Version Notes may be read independently when investigating a particular version.

For a complete understanding of the project's evolution, they are best read sequentially, as each document records the incremental changes from the preceding version.

---

## Naming Convention

Each Version Note follows the naming convention:

```text
README-v<major>.<minor>.<patch>.md
```

Examples:

```text
README-v3.2.5.md
README-v3.2.6.md
README-v3.2.7.md
```

Each document represents the engineering work completed for that development version.

---

*These Version Notes describe the evolution of Lumen during development. They are not release notes and should not be interpreted as indicating public or production releases.*
