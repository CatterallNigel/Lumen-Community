# Engineering Diary — From Proof of Concept to MVP

**Date:** 21–22 August 2026  
**Project:** Lumen

## Context

The last two days have marked a significant change in Lumen's development.

This has not primarily been a period of adding features. Instead, it has been a period of examining what already exists, identifying assumptions that were acceptable during proof-of-concept development, and turning those assumptions into explicit architecture, requirements, limitations and acceptance criteria.

The distinction is important.

Lumen was already a well-defined proof of concept. The architecture, service responsibilities and research direction were increasingly coherent. What changed was the prospect of placing Lumen into the hands of someone outside its development environment.

That changed the engineering standard.

> **The objective is no longer merely to demonstrate that Lumen can work. The objective is to define precisely what Lumen M0.1 is, close the gaps necessary for someone else to use it, and establish objective criteria by which we can say that release is ready.**

## External Perspectives

Recent external discussions have continued to influence the way Lumen's role is understood without changing its fundamental direction.

Daniel's work around consequential trust reinforced the importance of evidence, accumulated confidence and the question of what evidence eventually permits us to stop re-verifying every individual result.

That aligns closely with Lumen's developing Reasoning Assurance position: trust should not be treated as the absence of verification, but as something that can emerge from accumulated, inspectable evidence.

Discussions around Flow and provenance have also helped sharpen an important distinction.

Provenance is not necessarily one universal problem with one universal solution. Different systems can provide provenance within different domains. Lumen's concern is the provenance of model execution, reasoning conditions, behavioural reproduction and assessment. Other systems may establish provenance elsewhere in the information or operational chain.

Those approaches can therefore be complementary rather than competitive.

This continues a useful pattern in Lumen's development: external perspectives are increasingly helping identify the boundaries of the problem rather than simply supplying features.

## Monica and the External Researcher

Monica has become the immediate impetus for the next engineering stage.

The important point is not that Monica has changed Lumen's direction or supplied a new specification. It is that there is now a credible external researcher who may actually install the system, use it independently, and evaluate it without the assumptions and knowledge available inside the development environment.

> **Monica did not change the direction of Lumen; the prospect of putting Lumen into Monica's hands changed the standard to which the existing direction had to be engineered.**

That immediately exposed questions that could remain implicit while Lumen was being operated by its developer:

- What exactly constitutes the first external release?
- What is deliberately not included?
- How is the installation licensed?
- How is an authorised installation distinguished from a copied distribution?
- What happens if Servire is opened by another user?
- Are concurrent sessions genuinely isolated?
- Does each Replay begin with a clean session?
- Is the original execution context reproduced exactly?
- What happens to system prompts when Moderari replaces them?
- Can a Replay accidentally send two system prompts?
- How do control commands receive acknowledgements and responses?
- Which service owns a command?
- How is shared command state distributed without unnecessary noise?
- What belongs in operational logging, Nuntius diagnostics and Vestigare Trace?
- How are service configurations protected in an externally distributed source system?
- What constitutes acceptance rather than simply apparent functionality?

These are not new product ambitions. They are gaps between a successful proof of concept and an independently usable product.

## M0.1 as a Release Boundary

M0.1 is now being treated as Lumen's first deliberately bounded external research release.

The documentation has consequently become part of the engineering process rather than a record written after development.

The release now has:

- explicit development requirements;
- a development roadmap;
- requirements and limitations;
- acceptance conditions;
- release-level security architecture;
- research licensing position;
- service-specific development documents;
- cross-service requirements;
- a Future Development register.

This provides a simple scope rule:

> **Required to make the existing capability coherent, reproducible, secure enough for its stated distribution model, and independently usable belongs in M0.1. Valuable new capability that is not required to achieve that belongs in Future Development.**

That rule has already proved useful.

Ideas such as richer human answer ratings, multiple tool-provider development, bounded-resource behavioural research, experiment-scoped Moderari policy and release/update notification through the authorization heartbeat remain valuable, but they no longer automatically expand M0.1.

They are recorded rather than lost, while the current release remains bounded.

## Nuntius and the Lumen Control Plane

One of the larger architectural developments has been the definition of Lumen Nuntius.

The `\obt` mechanism is evolving from a collection of service-specific commands into a Lumen control-command language.

Rather than individual services inventing private command paths, Pontis can route `\obt` commands into Nuntius, which distributes them to the services currently known to be available.

Servire provides the bootstrap service catalogue and can update Nuntius as services start and stop.

The work has also forced clearer command semantics.

Commands may represent execution with acknowledgement, or requests that return data. Responses must be correlated with their origin so Pontis can return user-requested results to the correct session, while shared results can be distributed according to service configuration.

This has led to a useful separation of evidence:

> **Servire Ops log = what matters operationally to the Lumen operator.**

> **Nuntius diagnostics = what happened inside the control-command transport.**

> **Vestigare Trace = what happened in the model execution.**

That separation should prevent internal polling and control traffic from overwhelming the operational view while retaining the diagnostic evidence required to understand failures.

## Replay Fidelity and Session Isolation

Reviewing Replay for external use has exposed several assumptions that now need to become explicit invariants.

A Replay must not simply reproduce an ask. It must reproduce the execution conditions that materially affected that ask.

Each Replay execution must therefore begin in a new isolated session.

Repeated runs must look conceptually like:

```text
Source Trace T-001

Run 001 -> Session R-001
Run 002 -> Session R-002
Run 003 -> Session R-003
```

and never progressively reuse the context generated by preceding runs.

Otherwise repetition becomes continuation, and an Experiment is contaminated by its own previous observations.

The existing implementation must first be inspected to determine whether it already behaves correctly. Regardless of the result, session isolation is now an explicit M0.1 requirement.

## System-Prompt Fidelity

System-prompt handling has revealed another potentially important POC assumption.

Moderari is being extended to support three explicit policies:

```text
Pass-through
Moderari Default
Custom
```

Custom prompts can be saved, selected, edited and deleted for reuse.

Replay requires the original effective execution condition, however, not merely whichever system prompt is currently configured in Moderari.

This exposed a subtle question in Vestigare.

A client may submit:

```text
system = USER-A
```

while Moderari, operating in Custom mode, replaces it with:

```text
system = CUSTOM-X
```

The model therefore sees `CUSTOM-X`.

Both facts matter, but they have different meanings.

The Trace should preserve the incoming client prompt as provenance while identifying `CUSTOM-X` as the effective system prompt that reached the model. From Repetere's perspective, the superseded `USER-A` prompt is effectively soft-deleted: it remains visible evidence but must not be reconstructed as active model context.

Otherwise a Replay could potentially send:

```text
system = USER-A
system = CUSTOM-X
```

and still appear to match on simple tests where the duplicate context does not materially alter the answer.

A specific Vestigare investigation has therefore been defined for M0.1:

1. inspect a fresh baseline Trace;
2. inspect a Pass-through Trace;
3. inspect a Custom Trace;
4. compare what Vestigare actually records;
5. inspect what Repetere reconstructs;
6. use deliberately distinguishable system-prompt instructions so an incorrect reconstruction is behaviourally visible.

Only after that evidence is available will Vestigare remediation be implemented if required.

A candidate mechanism is for Moderari to notify Vestigare, through Nuntius, of the effective prompt whenever Moderari replaces the incoming prompt. The original remains provenance; the replacement becomes the sole active Replay system prompt.

The broader principle is:

> **Trace records both what entered Lumen and what actually reached the model when those differ, without confusing provenance with replayable execution context.**

## Runtime Authorization and Code Protection

Preparing a source-distributed research release has also required a much more explicit security boundary.

The central distinction is now:

> **Possession of the Lumen distribution is not authorization to operate Lumen.**

Servire becomes the local operational root of trust for the installed Lumen service group, while Illuminates.One remains the authority that grants runtime authorization.

The emerging M0.1 design includes:

- a Distribution ID;
- a Service Group UUID;
- an installation-specific cryptographic identity;
- TLS/HTTPS communication with Illuminates.One;
- cryptographically authenticated authorization;
- signed authorization responses;
- nonce/freshness protection;
- a time-limited authorization lease;
- bounded grace for genuine connectivity failure;
- eventual loss of authorised operation when authorization cannot be renewed;
- Servire-controlled protected-service startup;
- protected service configuration.

The Service Group UUID is explicitly an identifier, not a secret.

The architecture also recognises its limit. Lumen is being distributed as source and will execute on researcher-controlled hardware. Absolute tamper resistance is therefore not a credible objective.

The goal is instead to prevent casual copying from creating another authorised installation, make circumvention deliberate, retain installation provenance, and reduce dependence on legal enforcement alone.

## Protected Configuration

Security work has also exposed a cross-service concern.

Each service continues to own the meaning and validation of its own configuration, but protected operational configuration should not simply be distributed as persistent plaintext beside the service.

A common cross-service configuration contract is now required.

Configuration will need to be classified rather than blindly encrypted:

```text
Public / Non-Protected
Protected
```

Protected configuration should be encrypted at rest and made usable through the authorised runtime boundary without distributing a permanent master unlock secret beside the source code.

Where practical, decrypted protected values should exist only in process memory and should not leak into logs, health responses, Nuntius diagnostics or Vestigare Trace.

This also forces useful future-proofing. Praebere's configuration model, for example, should not assume that every provider behaves like local Ollama; future providers may introduce credentials and protected endpoints.

## Research Licensing

The Research licensing position has been brought into alignment with the technical architecture.

The licence and the runtime controls have different responsibilities.

The licence defines what the researcher is permitted to do. The runtime architecture defines how an authorised installation is technically recognised and operated.

The Research entitlement, Service Group UUID and installation cryptographic identity are therefore distinct concepts.

The licence also preserves an important research principle: runtime protection exists to enforce the distribution/licensing boundary, not to prevent legitimate observation, experimentation, criticism or publication of research findings.

Each release will retain the licensing position under which it was issued. The current licensing documentation may evolve, but the licence snapshot associated with a released version becomes part of that release's provenance.

This will also allow future updates to carry revised licensing terms where necessary, with explicit acceptance rather than silently changing the basis under which an existing release operates.

## Future Development as Scope Control

A Future Development register has now been introduced.

This is deliberately not a roadmap.

Detailed future design remains with the service or research area to which it belongs. The register contains only a short description and a pointer to that detailed material.

When a future release is planned, the register can be reviewed and selected items promoted into that release's requirements, roadmap and acceptance criteria.

One example already recorded is Fiducia's future ability to establish a Moderari execution policy once for an entire Experiment rather than requiring every individual Replay to repeat the same policy change.

Another is using the runtime authorization heartbeat to report future release availability.

That mechanism might eventually distinguish:

```text
AVAILABLE
RECOMMENDED
REQUIRED
```

and indicate where an upgrade carries revised licensing terms.

The mechanics remain deliberately undefined.

The important development discipline is:

> **A useful idea can be preserved without becoming current-release scope.**

## External Review as Validation

Providing Monica with the M0.1 documentation before providing the software has value beyond keeping her informed.

It gives an external researcher visibility into:

- what is being built;
- why it is being built;
- what the release is intended to support;
- what it deliberately does not support;
- what limitations are known before she receives it.

It also gives her an opportunity to challenge those assumptions before implementation is frozen.

That creates two-way confidence.

Monica can see that the research distribution is being deliberately engineered rather than packaged opportunistically. Lumen gains an external view of whether its architecture, scope and research proposition are understandable to somebody who was not involved in creating them.

The progression becomes:

```text
Document Review
      |
      v
Scope Validation
      |
      v
Implementation
      |
      v
Independent Use
      |
      v
Research Feedback
```

That is considerably stronger validation than simply delivering a Docker image and asking whether it works.

## Reflection

The proof-of-concept phase was necessarily exploratory.

Build something. Observe what it implies. Adjust the architecture. Discover another boundary. Build again.

M0.1 changes that mode of development.

The work is now increasingly organised around explicit dependencies, requirements and acceptance conditions. Gaps are being closed rather than features accumulated. Interesting ideas are being recorded without automatically becoming current work.

In that sense, the last two days have been less about expanding Lumen and more about **making Lumen finite**.

That is an important transition.

A proof of concept can succeed because its creator understands its assumptions.

An MVP has to succeed when somebody else does not.

> **M0.1 is becoming deliberately less ambitious and considerably more rigorous.**

That is the standard required before Lumen leaves the development environment and becomes something another researcher can independently install, operate, examine and challenge.
