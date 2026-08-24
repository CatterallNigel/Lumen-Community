# Lumen Servire — M0.1 Runtime Authorization and Security Development

**Release:** Lumen M0.1  
**Component:** Lumen Servire  
**Status:** Development Requirement  
**Related:** `M0.1_RUNTIME_AUTHORIZATION_AND_CODE_PROTECTION.md`

## 1. Purpose

For the External Research Distribution, Servire becomes more than the Lumen operational console and service lifecycle manager.

Servire is the **runtime authorization authority for the local Lumen installation**.

The governing rule is:

> **Illuminates.One authorises the installation. Servire establishes, maintains and enforces that authorization locally. Individual Lumen services trust Servire rather than independently implementing the external authorization relationship.**

This document identifies the Servire-specific changes required to implement that boundary for M0.1.

## 2. Responsibilities Added to Servire

Servire must become responsible for:

- installation and distribution identity;
- protection and use of the installation cryptographic identity;
- authorization communication with Illuminates.One;
- verification of Illuminates.One authorization;
- authorization lease state and renewal;
- authorization-aware stack lifecycle;
- authorised launch of protected Lumen services;
- protected runtime-configuration release;
- security-state presentation and diagnostics.

These responsibilities belong in Servire because Servire already owns the operational lifecycle of the Lumen service group.

## 3. Installation Identity

Servire must know the identity of the distribution and installation it is operating.

At minimum:

```text
distribution_id
service_group_uuid
installation_key_id
```

The `service_group_uuid` identifies the installation but is not an authentication secret.

Each externally provisioned installation must additionally have an asymmetric cryptographic identity.

Conceptually:

```text
Installation
    |
    +-- Distribution ID
    +-- Service Group UUID
    |
    +-- Installation Private Key
            |
            +-- retained/protected locally

Illuminates.One
    |
    +-- Service Group UUID
    +-- Distribution ID
    +-- Installation Public Key
```

Servire must never transmit the installation private key during normal authorization.

## 4. Installation-Key Storage

Servire owns access to the installation private identity.

Preferred storage order is:

1. hardware/TPM-backed storage where practical;
2. operating-system credential/key storage;
3. protected encrypted local storage.

The key must not be treated as an ordinary plaintext Servire configuration value.

M0.1 must document which storage mechanism is actually implemented so the protection provided by the release is not overstated.

## 5. Illuminates.One Trust Material

Servire requires the public verification material needed to authenticate authorization issued by Illuminates.One.

The corresponding Illuminates.One private signing key must never be distributed with Lumen.

Conceptually:

```text
Illuminates.One
    PRIVATE authorization signing key
            |
            +-- never distributed

Servire
    PUBLIC authorization verification key
            |
            +-- distributed for verification
```

Possession of the public key permits verification only.

## 6. Authorization Client

Servire must implement an HTTPS authorization client for the Illuminates.One authorization endpoint.

A request should contain the information required to establish installation identity, release provenance and freshness, for example:

```text
service_group_uuid
distribution_id
servire_version
release_version
timestamp
nonce
installation_key_id
service_manifest_hash   # if implemented
```

Only information required for authorization and operational provenance should be transmitted.

No unnecessary researcher-identifying information should be introduced.

## 7. Request Authentication

Before transmission, Servire must cryptographically authenticate the authorization request using the installation private identity.

Illuminates.One must therefore be able to establish:

> **This request was made by the cryptographic installation identity registered for this Service Group UUID and Distribution ID.**

The UUID alone must never satisfy this test.

## 8. Authorization Response Verification

Illuminates.One returns a signed authorization assertion.

Servire must verify, at minimum:

- the Illuminates.One signature;
- the expected installation/Service Group identity;
- the expected distribution where included;
- the request nonce/freshness binding;
- issue and expiry information;
- any authorization status/capability information used by M0.1.

A response that cannot be cryptographically verified must never extend the current authorization lease.

## 9. Nonce and Replay Protection

Every fresh authorization request must include a cryptographically secure nonce or an equivalent established freshness mechanism.

The authorization response must be bound to that challenge.

Servire must reject a previously valid response when it does not correspond to the current authorization request.

This prevents a recorded successful heartbeat/authorization exchange from being used as a reusable authorization credential.

## 10. Authorization Lease

Authorization is a lease rather than a permanent state.

Servire should expose an internal state model similar to:

```text
STARTING
    |
    v
AUTHORIZING
    |
    +---- rejected/invalid ----> UNAUTHORISED
    |
    v
AUTHORISED
    |
    v
RENEWAL_DUE
    |
    +---- renewal succeeds ----> AUTHORISED
    |
    v
GRACE
    |
    +---- renewal succeeds ----> AUTHORISED
    |
    v
EXPIRED
```

The exact lease and grace durations should be policy/configuration supplied by Illuminates.One rather than permanently embedded as product constants.

## 11. Startup Behaviour

For the external distribution, Servire must establish sufficient valid authorization before protected Lumen configuration and services become available for normal operation.

Conceptually:

```text
Servire starts
    |
    v
load installation identity
    |
    v
contact Illuminates.One
    |
    v
authenticate + authorize
    |
    +-- failure ----------> remain unavailable / report state
    |
    v
authorization lease established
    |
    v
unlock required runtime configuration
    |
    v
validate dependencies
    |
    v
authorise/start Lumen services
```

The UI must make authorization failure distinguishable from ordinary service-start failure.

## 12. Runtime Renewal

Servire must renew the authorization lease periodically while the protected stack is operating.

Temporary network or Illuminates.One availability failures must not immediately destroy legitimate research work.

The lease/grace mechanism therefore provides bounded tolerance.

However:

> **Loss or deliberate suppression of the authorization heartbeat must not permit indefinite operation.**

When the permitted lease/grace period expires, Servire must enforce the release policy and prevent continued normal authorised operation.

The exact shutdown/disable sequence should be explicit and deterministic so services do not remain in an ambiguous partially authorised state.

## 13. Service Launch Authorization

Starting a protected service directly by discovering its Python entry point, executable, HTTP command or container command must not constitute normal authorised M0.1 startup.

Servire must issue a short-lived service-start authorization.

Candidate mechanisms include:

- short-lived launch tokens;
- runtime credentials;
- protected inherited environment state;
- authenticated local control channels;
- signed or authenticated Servire launch assertions.

The final mechanism is an implementation decision.

It must be:

- tied to the current installation;
- tied to the intended service;
- short-lived;
- unsuitable as a permanent reusable credential.

## 14. Service Authority Boundary

A protected service may establish that Servire authorised its launch.

It must not thereby acquire Servire's authority.

In particular:

> **A Lumen service must not receive enough authorization material to authorise or start another protected Lumen service independently.**

Services needing lifecycle changes should request them through the existing Servire control architecture.

## 15. Runtime Configuration Release

Servire participates in making protected service configuration usable after authorization.

Servire must not simply decrypt every service configuration into permanent plaintext files.

The intended pattern is:

```text
encrypted service configuration
            +
valid runtime authorization
            +
installation-specific material
            |
            v
authorised runtime configuration
            |
            v
protected service
```

The exact cryptographic/key-envelope mechanism is defined separately because it applies across the Lumen service family.

Servire's responsibility is to enforce the authorization side of that mechanism and provide the resulting runtime material to the intended service through an appropriately protected path.

## 16. UI Requirements

Servire should expose the installation authorization state without overwhelming the normal operational interface.

At minimum the operator should be able to determine:

```text
Installation
    Service Group: <identifier>
    Distribution:  M0.1 / <distribution-id>

Authorization
    State:         AUTHORISED
    Lease expiry:  <time>
    Last renewal:  <time>
```

Useful failure states should distinguish:

- Illuminates.One unreachable;
- authorization rejected;
- installation identity invalid/unrecognised;
- authorization signature invalid;
- lease in grace;
- authorization expired;
- protected configuration unavailable;
- service launch authorization rejected.

Security diagnostics should be detailed enough for support without exposing private key material or reusable secrets.

## 17. Logging

Authorization events are operationally significant and belong in Servire's operational evidence.

Appropriate events include:

- authorization established;
- renewal succeeded;
- renewal temporarily failed;
- grace entered;
- authorization rejected;
- authorization cryptographically invalid;
- authorization expired;
- protected stack disabled/stopped because authorization expired;
- service launch authorization failed.

Logs must never include:

- installation private-key material;
- configuration decryption keys;
- complete reusable authorization credentials;
- other secrets capable of reproducing authority.

## 18. Development / External Distribution Separation

Developer operation and external research operation may require different startup behaviour.

Development builds may retain direct service execution where required for engineering and testing.

The external M0.1 distribution must not rely on that development convenience as an authorised runtime path.

The distinction should be explicit rather than implemented through an undocumented bypass secret.

## 19. Failure Behaviour

Security failures should fail closed where authorization cannot be established or verified.

This does not mean every transient network failure immediately stops the stack; the lease provides the permitted tolerance.

Examples:

```text
Illuminates.One temporarily unreachable
    -> retain existing valid lease
    -> retry
    -> enter grace if required

Invalid Illuminates.One signature
    -> reject response
    -> do not renew lease

Unknown/revoked installation
    -> authorization rejected

Expired lease + expired grace
    -> protected runtime no longer authorised

Invalid service launch assertion
    -> protected service refuses authorised startup
```

## 20. M0.1 Acceptance Tests

Servire security development is complete for M0.1 when tests demonstrate:

- a provisioned installation can establish authorization;
- UUID possession alone cannot authenticate an installation;
- the installation request is cryptographically attributable to the registered installation;
- Servire verifies Illuminates.One authorization;
- stale/replayed authorization responses are rejected;
- mismatched installation/distribution responses are rejected;
- invalid signatures are rejected;
- lease renewal succeeds normally;
- temporary connectivity failure follows defined grace behaviour;
- expired authorization prevents indefinite normal operation;
- deliberate heartbeat loss ultimately produces the same expiry behaviour;
- protected service startup requires valid Servire authorization;
- launch authorization is service- and installation-scoped;
- one service cannot use its authorization to authorise another;
- protected configuration is released only through the authorised runtime mechanism;
- sensitive key/authorization material is not written to logs;
- a copied distribution does not become authorised merely through possession of its UUID and release files.

## 21. Non-Goals

This work does not attempt to provide:

- absolute DRM;
- absolute tamper resistance on researcher-controlled hardware;
- the complete commercial licensing system;
- commercial capability tiers;
- offline licences;
- final hardware migration;
- complete key-rotation/revocation infrastructure unless required by the chosen M0.1 provisioning design.

Those capabilities may build upon this boundary later.

## 22. Governing Principle

> **Servire is the local operational root of trust for an authorised Lumen installation, but the authority to grant that authorization remains external to the distributed code and under Illuminates.One control.**
