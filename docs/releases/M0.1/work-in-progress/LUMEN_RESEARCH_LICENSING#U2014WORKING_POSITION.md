# Lumen Research Licensing — Working Position

**Status:** Research / Working Draft  
**Purpose:** Define the intended licensing and distribution position for the external Lumen Research distribution before preparation of formal licence terms.

---

## 1. Purpose

The Lumen Research licence is intended to make Lumen available for genuine independent research, experimentation, evaluation and education while protecting the commercial development of the Lumen platform.

The Research distribution should be sufficiently complete to allow meaningful investigation of Lumen and its behaviour.

It is not intended to be a time-limited demonstration or an artificially crippled version of the software.

Restrictions should therefore correspond wherever possible to genuine distinctions between research use and commercial deployment rather than arbitrary feature limitations.

This document records the intended licensing position.

It is **not yet the legal licence agreement**.

---

## 2. Permitted Use

The Lumen Research distribution may be used for:

- Non-commercial research.
- Academic research.
- Independent technical evaluation.
- Experimentation with Lumen.
- Educational use.
- Evaluation of Lumen architecture and behaviour.
- Development of experiments, benchmarks and research methodologies using Lumen.
- Publication and discussion of research performed using Lumen.

Researchers must be free to publish legitimate findings arising from their use of Lumen, including:

- positive findings;
- negative findings;
- benchmark results;
- behavioural observations;
- comparisons;
- limitations;
- criticism.

The Research licence must not attempt to control or suppress legitimate research conclusions.

---

## 3. Commercial Use

The Research distribution may not be used for commercial or production purposes.

This includes using the Research distribution as infrastructure supporting a commercial product or service.

A Research installation may not itself be provided as a paid service to third parties.

The precise boundary between research conducted within a commercial organisation and commercial operational use requires further consideration before the licence is finalised.

---

## 4. Deployment Restriction

A Research installation is restricted to **one machine**.

All Lumen services forming that Research installation must execute on the same physical or virtual machine.

Lumen services may not be distributed across multiple machines under the Research licence.

This restriction applies to the Lumen service topology, not merely to the number of copies of individual binaries or repositories.

Distributed Lumen deployment is reserved for commercial licensing.

The precise treatment of external model providers, external tools and other systems consumed by Lumen should be explicitly defined before the licence is finalised. They should not inadvertently be interpreted as Lumen services merely because the Research installation communicates with them.

---

## 5. Included Lumen Capabilities

The Research distribution should contain the core Lumen capabilities necessary to perform meaningful research and experimentation.

The definitive component list should be established against the external Research distribution before release rather than permanently embedded in the licence where normal architectural evolution could make such a list obsolete.

The Research distribution does **not** include Lumen Aestimare.

Aestimare and its assessment engines are reserved for commercial Lumen distributions unless a future research-specific Aestimare licence is explicitly introduced.

---

## 6. Research Registration

Access to the Lumen Research distribution requires registration through the official Lumen website operated by Illuminates.One.

Registration should remain deliberately lightweight.

At minimum, registration requires a valid and active email address capable of receiving the Research distribution link.

The purpose of registration is to:

- establish an authorised Research distribution entitlement;
- provide access to the official Research distribution;
- associate the distribution with its unique identifier;
- support legitimate re-downloads;
- support licence validation; and
- provide a reliable mechanism for important distribution or licensing communications.

Registration should not require unnecessary personal information.

---

## 7. Official Distribution

The Lumen Research distribution must be obtained directly from Illuminates.One through the official Lumen Research distribution mechanism.

After successful registration, Illuminates.One will send the registered email address an authorised download link.

The download link provides access to the Research distribution associated with that registration.

Research copies obtained from another person, organisation, repository, mirror, download service or other unofficial source are not authorised Research distributions.

---

## 8. Research Distribution Identity

Each authorised Research registration will be associated with a unique UUID or equivalent distribution identifier.

The identifier belongs to the authorised **Research distribution entitlement**, rather than to an individual download event.

Consequently, repeated legitimate downloads using the authorised download mechanism should receive a Research distribution associated with the **same UUID**.

For example:

```text
Researcher registers
        │
        ▼
Illuminates.One creates Research entitlement
        │
        ├── Registered email
        └── Research UUID
                │
                ▼
        Download link issued
                │
        ┌───────┴────────┐
        ▼                ▼
   First download    Later re-download
        │                │
        └───────┬────────┘
                ▼
          Same Research UUID
```

This allows a researcher whose local copy has been lost, deleted, corrupted or otherwise destroyed to obtain the authorised distribution again without requiring a new Research entitlement.

The Research distribution entitlement identifier is not, by itself, proof that a particular installation is authorised to operate.

An installed Research distribution may additionally be provisioned with a **Service Group UUID** identifying the installed Lumen service group and an installation-specific cryptographic identity used for runtime authorization.

These identifiers have different purposes:

```text
Research distribution entitlement
    identifies the authorised distribution entitlement

Service Group UUID
    identifies the installed Lumen service group

Installation cryptographic identity
    authenticates the installation during runtime authorization
```

Neither the Research UUID nor the Service Group UUID should be treated as a secret or as sufficient proof of authorization by itself.

The detailed generation, provisioning, signing, validation and protection mechanisms are technical implementation matters defined by the M0.1 runtime-authorization architecture rather than by this licensing-position document.

---

## 9. Redistribution

The Lumen Research distribution may **not be redistributed**.

A Research licence holder may not:

- provide their Research distribution to another person or organisation;
- publish or host the Research distribution for download;
- mirror the Research distribution;
- distribute copies of the Research package;
- incorporate the Research distribution into another distributable software package;
- incorporate it into a distributable container image;
- incorporate it into a distributable virtual-machine image;
- make the distribution available through a public or private software repository for use by others; or
- otherwise provide another party with a copy obtained through their Research entitlement.

A person wishing to use Lumen Research must register independently and obtain an authorised Research distribution directly from Illuminates.One.

This restriction applies to the **Lumen Research software distribution itself**.

It does not restrict legitimate publication or sharing of:

- research results;
- papers;
- benchmarks;
- observations;
- experimental methodologies;
- comparisons;
- criticism; or
- other findings produced through permitted use of Lumen.

---

## 10. Re-download and Recovery

An authorised Research user may use their issued download mechanism to recover their Research distribution.

A re-download is not redistribution and does not create a new Research entitlement.

The re-downloaded distribution should retain the same Research UUID associated with the original registration.

This mechanism is intended to cover ordinary circumstances such as:

- accidental deletion;
- storage failure;
- machine replacement;
- corrupted downloads;
- lost local installation media; and
- deliberate reinstallation.

Whether the same Research entitlement may be **simultaneously active on more than one machine** remains a separate licensing decision.

The existence of a re-download mechanism must not be interpreted as permission for multiple concurrent Research installations.

---

## 11. Updates

Research users may receive periodic updates to the Research distribution.

Research updates do not necessarily follow the same release schedule as commercial releases.

No commitment is presently made that every internal or commercial Lumen release will result in an equivalent Research release.

The registered distribution mechanism may ultimately also provide access to authorised Research updates.

The supported update mechanism, compatibility policy and treatment of older Research versions remain to be determined.

---

## 12. Runtime Authorization and Operational Heartbeat

Operation of the Research distribution requires continuing runtime authorization with Illuminates.One.

The operational heartbeat forms part of that runtime-authorization mechanism and is a condition of the Research licence.

Possession of the Research source code, distribution package, Research UUID, Service Group UUID or installed files does **not**, by itself, constitute authorization to operate a Research installation.

A Research licence holder may not deliberately:

- disable;
- remove;
- block;
- bypass;
- falsify;
- replay;
- substitute;
- interfere with; or
- modify for the purpose of defeating

the runtime-authorization, heartbeat, installation-identity or licence-validation mechanisms.

The Research installation is expected to authenticate as the installation associated with its authorised entitlement. The technical implementation may use installation-specific cryptographic identity, signed authorization assertions, freshness protection and time-limited authorization leases.

The detailed cryptographic design is an implementation/security matter and should not be duplicated in the eventual legal licence except where necessary to express the licence holder's obligations.

Deliberately disabling or circumventing the required runtime-authorization/heartbeat mechanism will ultimately cause the Lumen Research installation to become unavailable for authorised operation.

This behaviour is an intentional licensing control.

The Research UUID and Service Group UUID may participate in identification and validation, but neither is intended to function as a standalone authentication secret.

---

## 13. Heartbeat Data Principle

The runtime-authorization heartbeat must follow a **minimum necessary information** principle.

Illuminates.One should receive only information required to:

- identify the authorised Research distribution entitlement;
- identify the installed Lumen service group;
- authenticate/validate the authorised installation where required;
- establish that it is a Research installation;
- validate licence status;
- determine relevant software/distribution versions;
- validate permitted capabilities; and
- operate the licensing mechanism.

The heartbeat must not be designed as a mechanism for collecting research activity.

In particular, the heartbeat should not transmit:

- user prompts;
- model responses;
- conversation contents;
- Trace contents;
- Replay contents;
- research results;
- user documents;
- model-generated content;
- research datasets; or
- other substantive user or research data.

The final heartbeat data schema must be documented before external Research distribution.

---

## 14. Heartbeat Data Retention

Raw runtime-authorization/heartbeat information should be retained for no longer than **30 days** unless a shorter operational period proves practical.

Retention should be limited to what is necessary for licensing, security, reliability and reasonable operational diagnosis.

Whether anonymised or aggregated operational statistics may be retained beyond the raw-data retention period requires a separate explicit decision.

This must not become an implicit mechanism for retaining identifiable Research usage indefinitely.

---

## 15. Connectivity Failure

Intentional runtime-authorization circumvention and genuine connectivity failure are different conditions.

A temporary inability to contact Illuminates.One should not immediately disable a legitimate Research installation.

The implementation therefore uses a **time-limited authorization lease with a bounded grace mechanism** so that temporary network or Illuminates.One outages can be tolerated without permitting indefinite offline operation.

The precise lease duration, renewal interval and grace period remain operational/security policy and do not need to be permanently fixed in the licence text.

Once the valid authorization lease and permitted grace state have expired, the Research installation must no longer continue normal authorised operation.

The objective is:

> tolerate genuine connectivity failure without making deliberate long-term authorization avoidance possible.

---

## 16. Circumvention

The Research licence should prohibit modifications whose purpose is to defeat Research licence restrictions.

This includes deliberate circumvention of:

- runtime authorization or heartbeat validation;
- distribution, Service Group or installation identity;
- cryptographic authorization or freshness controls;
- protected service-start controls;
- protected configuration controls;
- deployment restrictions;
- Research-only limitations;
- excluded capabilities; and
- other technical licensing controls.

This restriction should not be written so broadly that it unintentionally prevents legitimate observation, experimentation, interoperability work or research into Lumen itself.

That distinction requires careful treatment in the eventual legal licence.

---

## 17. Authorised Installation and Technical Controls

The Research licence authorises use of an appropriately provisioned Research installation; it does not grant a right to create additional authorised installations merely by copying the distributed source, containers, configuration or installed filesystem.

A licence holder may not copy, transfer, clone or reuse installation-specific credentials, cryptographic identity or other authorization material for the purpose of causing another installation to appear to be the authorised installation.

This restriction does not prohibit ordinary permitted backup, re-download, recovery or machine-replacement activity where performed through an Illuminates.One-supported mechanism.

Protected configuration, service-start authorization and other technical controls may be used to enforce the authorised installation boundary.

Researchers remain free to observe, test and report on Lumen within the rights granted by the Research licence. The purpose of these controls is to enforce the licensing boundary, not to prevent legitimate research findings or criticism.

---

## 18. Research Integrity

The Research licence should preserve the ability to investigate Lumen independently.

Licensing controls should protect the distinction between Research and Commercial distributions without undermining the usefulness or credibility of the Research distribution.

The intended principle is:

> **Research users receive a genuine Lumen environment with defined deployment, distribution and commercial-use restrictions, not a demonstration environment whose limitations invalidate meaningful research.**

---

## 19. Decisions Still Required

The following matters are deliberately unresolved and should not be hurried:

- Exact definition of commercial use.
- Treatment of research performed by commercial organisations.
- Whether multiple simultaneous Research installations are permitted for one entitlement.
- Definition of a "machine" for virtualisation, containers and cloud instances.
- Treatment of machine replacement under the same Research UUID.
- Exact boundary between a Lumen service and an external dependency/provider.
- Exact component set included in the first Research distribution.
- Download-link lifetime and recovery mechanism.
- Whether download links may be regenerated.
- Whether registration is attached to a person, email address or other Research identity.
- Distribution signing and integrity verification.
- Installation provisioning, recovery and machine-migration procedure.
- Installation identity key storage and rotation policy.
- Revocation and reinstatement procedure.
- Final runtime-authorization/heartbeat data schema.
- Runtime-authorization lease, renewal and grace durations.
- Research release/update policy.
- Version compatibility and support expectations.
- Whether aggregated/anonymised heartbeat statistics may be retained beyond 30 days.
- Whether modification of Lumen itself is permitted and, if so, under what conditions.
- Source availability and associated rights, where applicable.
- Treatment of derived works.
- Licence termination and reinstatement.
- Appropriate jurisdiction and governing law.
- Whether a future research-specific Aestimare offering should exist.

---

## 20. Current Position

The current intended Research proposition can therefore be summarised as:

**Registered Research use — direct Illuminates.One distribution — unique Research entitlement — authorised single-machine installation — re-downloadable but non-redistributable — core research capabilities — no Aestimare — periodic updates — mandatory privacy-minimising runtime authorization/heartbeat — protected installation controls — no commercial/production use.**

This summary is descriptive only.

The unresolved matters above must be considered before this position is converted into formal licence terms.