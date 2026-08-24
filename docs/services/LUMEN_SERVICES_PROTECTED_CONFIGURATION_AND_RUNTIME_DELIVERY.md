# Lumen Services — Protected Configuration and Authorised Runtime Configuration

**Release:** Lumen M0.1  
**Scope:** All protected Lumen services  
**Status:** Cross-Service Development Requirement  
**Related:** `M0.1_RUNTIME_AUTHORIZATION_AND_CODE_PROTECTION.md`

## 1. Purpose

The External Research Distribution introduces a security boundary that applies across the Lumen service family.

Protected operational configuration must not simply be shipped as ordinary plaintext configuration beside each service, and possession of the distributed source/configuration must not by itself provide everything required for normal authorised operation.

This document defines the common cross-service configuration contract.

It deliberately separates:

- **configuration ownership** — each service still owns the meaning and validation of its own configuration;
- **configuration protection** — protected distributed values are encrypted at rest;
- **runtime authorization** — Servire establishes whether the installation is currently authorised;
- **runtime delivery** — the intended service receives usable configuration only through the authorised runtime mechanism.

## 2. Governing Rule

> **A distributed Lumen installation must not contain, in persistent plaintext form, all protected configuration and all permanent key material required to operate the protected stack independently of runtime authorization.**

Encryption is not useful if the complete permanent decryption key is distributed beside the ciphertext.

## 3. What This Does Not Change

This requirement does not move service-specific configuration semantics into Servire.

For example:

- Moderari still defines and validates Moderari configuration;
- Praebere still defines and validates provider/model configuration;
- Nuntius still defines and validates its own service configuration;
- Repetere, Vestigare, Pontis, Rogare and Fiducia retain their existing configuration responsibilities.

Servire is the authorization and runtime-release authority, not the owner of every service's configuration schema.

## 4. Configuration Classification

Not every configuration value needs encryption.

Each service should classify configuration into at least two conceptual categories.

### Public / Non-Protected Configuration

Values whose disclosure does not grant protected capability or expose sensitive operational information.

Examples may include:

```text
service name
service version
local port
non-sensitive UI defaults
public endpoint names
public verification keys
```

These may remain in normal distributed configuration where appropriate.

### Protected Configuration

Values whose disclosure, copying or independent use would weaken the intended runtime boundary.

Examples may include:

```text
provider credentials
protected external endpoints
runtime authorization-derived material
private service credentials
sensitive integration configuration
protected deployment-specific values
```

The exact classification must be made per service.

A value should not be encrypted merely to make the distribution look opaque. Protection should have a defined security purpose.

## 5. Persistent Representation

Protected configuration should be stored encrypted at rest.

Conceptually:

```text
service/
    config/
        public.yaml
        protected.enc
```

The filenames and format are implementation choices.

The important distinction is that the external release must not require a persistent plaintext protected configuration file to sit beside the service executable.

## 6. Key Principle

The distributed installation alone must not permanently contain everything required to decrypt protected configuration.

A possible architecture is:

```text
installation-specific material
            +
valid authorization-derived material
            |
            v
runtime configuration key / unwrap capability
            |
            v
decrypt protected configuration
```

This is an architectural pattern, not a mandate to invent a custom key-derivation algorithm.

The final implementation must use established cryptographic primitives and libraries.

## 7. Encryption Design

The exact encryption scheme requires an implementation security decision.

The design should prefer a standard envelope-encryption or authenticated-encryption pattern rather than proprietary cryptography.

The selected mechanism must provide confidentiality and integrity/authenticity of protected configuration.

The implementation must define:

- encrypted file/container format;
- algorithm and library;
- key generation;
- key derivation or key unwrapping;
- nonce/IV handling;
- integrity/authentication;
- versioning of the encrypted format;
- failure behaviour;
- future migration/rotation path.

No cryptographic key should be derived from predictable identifiers such as the Service Group UUID alone.

## 8. Servire Relationship

Servire establishes the installation's valid runtime authorization.

After successful authorization, Servire participates in obtaining or deriving the material required to make protected configuration usable.

Conceptually:

```text
Illuminates.One
      |
      | signed runtime authorization
      | + authorization-derived material
      v
   Servire
      |
      | installation-specific material
      | + valid authorization state
      v
runtime configuration release
      |
      v
intended Lumen service
```

Servire should not become a permanent plaintext configuration repository.

## 9. Service Runtime Contract

A protected service must be able to start with configuration supplied through the authorised runtime path.

The service should not require Servire to write a decrypted configuration file to its normal filesystem before startup.

Possible runtime-delivery mechanisms include:

- protected process environment;
- inherited file descriptor/pipe;
- authenticated local IPC;
- memory-backed temporary storage;
- short-lived protected runtime file where unavoidable.

The final common mechanism should be selected deliberately so services do not each invent incompatible solutions.

## 10. Plaintext Lifetime

Where practical, decrypted protected configuration should exist only in memory for the lifetime required by the service.

It should not be:

- written back over the encrypted source;
- cached permanently in plaintext;
- copied into logs;
- exposed through diagnostic endpoints;
- returned through health endpoints;
- included in Trace;
- included in Nuntius diagnostics.

If a temporary plaintext representation is technically unavoidable, its creation, permissions, lifetime and secure removal must be explicitly designed and tested.

## 11. Environment Variables

Environment variables are convenient but should not automatically be treated as a secure secret store.

If used for runtime delivery, the implementation must consider:

- process inspection;
- child-process inheritance;
- diagnostic dumps;
- container inspection;
- crash reporting;
- accidental logging.

They may form part of the implementation, but their security properties must be understood rather than assumed.

## 12. Containers

Dockerisation does not itself make configuration secret.

The external M0.1 container images must not embed:

- plaintext protected service configuration;
- Illuminates.One private signing material;
- permanent master decryption keys;
- hidden authorization bypass credentials.

Secrets supplied at runtime must not be baked back into derived images as part of normal operation.

## 13. Service Startup

Protected configuration and service-launch authorization are related but distinct.

A service should require both:

```text
valid Servire-authorised startup
            +
valid runtime configuration
            |
            v
service enters normal operational state
```

Possession of decrypted configuration alone should not be treated as proof that the service was validly launched.

Similarly, a valid launch assertion should not itself expose all protected configuration.

## 14. Configuration Validation

Once configuration has been delivered, each service remains responsible for validating its own schema and required values.

Security-related decryption success does not imply application-level configuration correctness.

Failure should distinguish where practical between:

```text
CONFIGURATION_UNAVAILABLE
CONFIGURATION_DECRYPTION_FAILED
CONFIGURATION_INTEGRITY_FAILED
CONFIGURATION_INVALID
LAUNCH_UNAUTHORISED
```

These states aid diagnosis without exposing protected content.

## 15. Configuration Changes

The M0.1 implementation must define how a legitimate configuration change becomes a new encrypted persistent configuration.

A safe conceptual workflow is:

```text
authorised operator changes configuration
        |
        v
service/Servire validates proposed values
        |
        v
protected values encrypted
        |
        v
encrypted persistent representation updated
        |
        v
service reload/restart through Servire if required
```

Plaintext editing files should not become the normal external-distribution workflow for protected values.

Development environments may retain simpler mechanisms where explicitly separated from the external release.

## 16. Saved System Prompts

Saved Moderari custom system prompts require an explicit classification decision.

They are part of researcher-defined experimental configuration and may contain information the researcher expects to remain local.

For M0.1 they should therefore be reviewed under this configuration model rather than automatically assumed to be public configuration.

Regardless of their storage protection, the **effective system prompt actually supplied to the model remains execution evidence** and must be preserved by Vestigare according to the M0.1 Trace requirements.

Configuration protection must not damage experimental provenance.

## 17. Provider Configuration

Praebere provider configuration is a particularly important candidate for protected storage because future providers may require credentials or sensitive endpoint configuration.

The common configuration mechanism should therefore avoid an Ollama-only assumption merely because the initial local provider may not require credentials.

## 18. Nuntius and Servire Topology Configuration

Not all topology information is necessarily secret.

Service names, command ownership and ordinary routing metadata may be non-protected configuration.

Any credentials or protected endpoints used to authenticate inter-service or external control relationships should be classified separately.

The security mechanism should protect sensitive values without making ordinary operational topology unnecessarily opaque.

## 19. Trace Boundary

Vestigare records model-execution evidence.

Protected configuration must not be copied into Trace merely because it was used by a service.

Only execution conditions that legitimately form part of the model evidence should appear there.

For example:

```text
provider = ollama
model = qwen2.5-coder:14b-32k
effective system prompt = <actual prompt>
```

may be legitimate execution provenance.

A provider API secret or configuration-unlock key is not.

## 20. Logging Boundary

No Lumen service should log protected configuration values or cryptographic key material.

Logs may record:

- configuration loaded successfully;
- protected configuration unavailable;
- decryption failed;
- integrity validation failed;
- configuration version/hash where safe;
- reload/restart required.

Logs must not record the protected plaintext merely to aid diagnosis.

## 21. Copy Behaviour

Copying a service directory, container or complete Lumen installation must not automatically provide another installation with independently usable protected configuration.

This requires the encryption/unlock mechanism to depend upon installation-specific and/or authorization-derived material rather than only files that are copied with the distribution.

## 22. Development Environment

Developers need to run individual services and inspect configuration while engineering Lumen.

M0.1 should therefore explicitly distinguish:

```text
Development Runtime
    engineering convenience permitted

External Research Runtime
    protected configuration contract enforced
```

This separation must be implemented through an explicit build/runtime mode or provisioning model.

It must not depend upon a secret hard-coded bypass value embedded in distributed source.

Tests must cover the external-runtime behaviour.

## 23. Cross-Service Implementation Strategy

Because this requirement applies to every protected service, M0.1 should define one shared implementation pattern before modifying services individually.

Recommended sequence:

1. inventory configuration used by each M0.1 service;
2. classify values as public/non-protected or protected;
3. choose the standard encrypted representation;
4. choose the standard runtime-delivery mechanism;
5. define the Servire/configuration-unlock interface;
6. implement a shared Lumen configuration/security library where doing so avoids duplicated cryptographic code;
7. migrate one service as the reference implementation;
8. validate startup, failure and logging behaviour;
9. migrate the remaining protected services;
10. inspect the final distribution for plaintext protected values and embedded permanent secrets.

A shared implementation is strongly preferable to each service independently implementing encryption.

## 24. Acceptance Criteria

The cross-service protected-configuration work is complete for M0.1 when:

- configuration for every included service has been inventoried and classified;
- protected values are encrypted at rest in the external distribution;
- public values remain usable without unnecessary encryption;
- the complete permanent decryption authority is not stored beside the encrypted configuration;
- valid runtime authorization contributes to unlocking protected configuration;
- the chosen cryptographic mechanism uses established primitives/libraries;
- encrypted configuration has integrity protection;
- services can consume configuration through the standard authorised runtime path;
- decrypted protected configuration is not routinely persisted as plaintext;
- secrets do not appear in logs, health responses, Trace or Nuntius diagnostics;
- containers do not embed protected plaintext or master unlock secrets;
- copied service/install files alone do not make protected configuration independently usable;
- configuration failure states are diagnosable without exposing protected content;
- development conveniences are explicitly separated from external-runtime behaviour;
- the final external distribution has been inspected for accidental plaintext protected configuration and embedded secrets.

## 25. Future Considerations

The common configuration architecture should allow later addition of:

- key rotation;
- configuration-key rotation;
- installation migration;
- TPM/hardware-backed key use;
- operating-system secret stores;
- capability-specific configuration;
- commercial deployment policies;
- secure remote configuration;
- signed configuration manifests;
- secure update mechanisms.

These are not all required for M0.1.

## 26. Architectural Principle

> **Services own the meaning of their configuration. Servire owns the authorised runtime boundary. Protected configuration is persisted encrypted, released only through that boundary, and never made secure merely by hiding a key beside the code that uses it.**
