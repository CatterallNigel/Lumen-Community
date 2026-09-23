# Lumen M0.1 Runtime Authorization and Code Protection --- Release Position

**Release:** Lumen M0.1\
**Status:** FINAL M0.1 Release Position --- release checks reconciled
2026-09-23\
**Date:** 2026-09-22\
**Scope:** External Research Distribution\
**Classification:** Internal release/security document --- not for
researcher distribution

------------------------------------------------------------------------

## Final release reconciliation --- 2026-09-23

The final M0.1 distribution test has now closed the remaining release
checks relevant to this document. The frozen Registry image was removed
from the test machine and pulled again after all prior M0.1 containers
and persistent volumes were removed. The fresh installation required
Research Licence acceptance, created fresh installation state, completed
signed registration/authorization, validated its external dependencies,
passed 83/83 Servire validation checks and started the managed Stack
successfully.

The active release authorization policy is a 24-hour lease, normal
renewal at approximately 6 hours, and approximately hourly retry
following temporary authorization-service unavailability.

This clean re-pull demonstrates that installation identity and licence
acceptance are not reusable state baked into the frozen image.
Distribution Registry authentication and Lumen runtime
registration/authorization remain separate controls.

## 1. Purpose

This document records the runtime authorization and code-protection
position actually adopted for the Lumen M0.1 External Research
Distribution.

It replaces the earlier **M0.1 Runtime Authorization and Code
Protection** architecture document. That document mixed early design
proposals, possible future mechanisms and M0.1 requirements that were
subsequently changed during implementation.

This document is therefore a release-position record rather than a
development plan.

The detailed implementation and evidence remain defined by the private
M0.1 authorization documents. This document deliberately records the
security boundary at a higher level and does not duplicate
implementation-sensitive material unnecessarily.

------------------------------------------------------------------------

## 2. Release Principle

The governing M0.1 principle is:

> **Possession of the Lumen M0.1 distribution does not, by itself,
> constitute authorization to operate Lumen.**

M0.1 combines:

-   a contractual Research Licence;
-   an issued registration key;
-   a persistent installation identity;
-   cryptographically signed authorization;
-   periodic authorization with Illuminates.One;
-   Servire-controlled Stack operation; and
-   distribution packaging intended to discourage casual copying,
    modification and unsupported execution.

These mechanisms provide a practical authorization and release-control
boundary.

They are not represented as absolute DRM or tamper resistance.

Software executing on hardware controlled by another party must
ultimately be assumed accessible to a sufficiently determined operator.

------------------------------------------------------------------------

## 3. M0.1 Distribution Boundary

Lumen M0.1 is distributed as a Docker-based, single-machine research
distribution.

The supported topology consists of:

-   the Lumen application container;
-   MongoDB;
-   HAProxy as the external ingress;
-   an external Ollama provider; and
-   Illuminates.One for registration and periodic licence authorization.

The Lumen application container contains Servire and the managed Lumen
Stack services.

M0.1 is licensed and supported as a single-machine Lumen installation.
Distributed Lumen deployment across multiple machines is outside the
M0.1 Research Licence.

Aestimare is not included in M0.1.

------------------------------------------------------------------------

## 4. Authorization Trust Boundary

The implemented authorization relationship is:

``` text
Illuminates.One
      │
      │ signed authorization decision
      ▼
   Licentia
      │
      │ transport / protocol broker
      ▼
    Servire
      │
      │ verifies and enforces authorization
      ▼
  Lumen Stack
```

Responsibilities are deliberately separated.

### Illuminates.One

Illuminates.One:

-   owns authorization policy and authorization decisions;
-   maintains researcher, registration, installation and authorization
    evidence;
-   binds a registered installation identity to its public key;
-   signs authorization decisions; and
-   may explicitly place an installation into a locked state.

### Licentia

Licentia:

-   provides the authorization transport/protocol path between Servire
    and Illuminates.One;
-   does not decide whether an installation is authorized; and
-   cannot manufacture a valid Illuminates.One authorization decision.

### Servire

Servire:

-   owns the local installation identity;
-   signs authorization requests using the installation private key;
-   independently verifies Illuminates.One authorization responses;
-   maintains the local authorization lifecycle;
-   gates Stack operation according to authorization state; and
-   remains the operational authority for the managed Lumen Stack.

Other Lumen services do not independently implement the external
Illuminates.One authorization relationship.

------------------------------------------------------------------------

## 5. Installation Identity

Each M0.1 installation has a persistent cryptographic identity.

The implemented installation identity uses Ed25519.

The installation private key remains local to the Lumen installation.

The corresponding public key is registered with Illuminates.One during
initial registration and becomes bound to that installation.

The private installation key is not transmitted to Illuminates.One.

The Illuminates.One private signing key is not distributed with Lumen.

Servire contains the trusted public verification material required to
verify Illuminates.One authorization decisions.

The security objective is mutual cryptographic attribution:

-   Illuminates.One can verify that an authorization request was signed
    by the registered installation identity; and
-   Servire can verify that an authorization decision was signed by
    Illuminates.One.

------------------------------------------------------------------------

## 6. Initial Registration

An issued M0.1 distribution contains the registration material required
for its intended researcher/install.

On first registration:

1.  Servire establishes or loads its persistent installation identity.
2.  Servire constructs and signs the initial registration request.
3.  Licentia transports the request to Illuminates.One.
4.  Illuminates.One validates the registration key and installation
    evidence.
5.  Illuminates.One binds the installation identity/public key to the
    registration.
6.  Illuminates.One returns a signed authorization decision.
7.  Servire independently verifies that decision before accepting
    authorization.

An unknown registration key may enter the registration process only when
it satisfies the defined registration-key format and Illuminates.One
accepts it according to its registration policy.

The registration key is registration material; it is not the continuing
cryptographic identity of the installation.

------------------------------------------------------------------------

## 7. Periodic Licence Authorization

Continued operation of the M0.1 Research Distribution requires periodic
licence authorization with Illuminates.One.

The normal authorization lifecycle is based on a time-limited
authorization lease rather than continuous connectivity.

The intended M0.1 operational values are:

-   Illuminates.One authorization lease: **24 hours**;
-   normal Servire renewal request: approximately **every 6 hours**; and
-   retry following temporary authorization-service unavailability:
    approximately **hourly**.

These are operational implementation values, not terms of the Research
Licence.

Temporary loss of connectivity does not necessarily terminate an
authorized installation immediately. Servire uses the implemented
lease/grace lifecycle to distinguish temporary unavailability from
expiry or an explicit authorization decision.

The final M0.1 release configuration has been checked: the release
values are a 24-hour lease, normal renewal at approximately 6 hours, and
approximately hourly retry following temporary authorization-service
unavailability.

------------------------------------------------------------------------

## 8. Authorization States

The implemented Servire authorization lifecycle includes:

``` text
UNAUTHORISED
AUTHORIZING
AUTHORISED
GRACE
EXPIRED
LOCKED
```

The lifecycle has been live-proven through authorization, temporary
failure, expiry and recovery paths.

In particular, M0.1 has demonstrated:

-   successful initial registration;
-   signed authorization renewal;
-   `AUTHORISED → GRACE → EXPIRED → AUTHORISED` recovery;
-   signed `LOCKED` handling; and
-   `LOCKED → AUTHORISED` recovery without restarting Servire.

A signed lock reason/message/contact supplied by Illuminates.One is
presented by Servire rather than independently recreated by the local
installation.

------------------------------------------------------------------------

## 9. Persistence

The installation cryptographic identity persists across normal Lumen
restarts.

Servire persists the signed evidence from successful initial
registration separately from the current authorization lease.

The current authorization lease is runtime state rather than a
permanently reusable local authorization credential.

Restarting Lumen therefore does not create a new installation identity
and does not make possession of old authorization material sufficient to
manufacture a new authorization decision.

------------------------------------------------------------------------

## 10. Stack Authorization and Lifecycle

Servire is the local runtime security gate for M0.1.

The single-container M0.1 architecture deliberately does **not**
introduce a second per-service external authorization protocol or
separate licence state into every managed Lumen service.

The governing operational rule is:

> **Protected Lumen services are launched and managed through an
> authorized Servire process inside the authorized Lumen distribution
> runtime.**

Servire controls normal Stack lifecycle operations and applies the
external authorization state at the Stack boundary.

Direct execution of individual service internals is not a supported
researcher runtime path.

This is intentionally simpler than the earlier proposed design in which
every service would receive and independently renew a separate
short-lived Servire-issued authorization token.

That earlier per-service lease model is **not the M0.1 release
architecture**.

------------------------------------------------------------------------

## 11. Information Used for Licence Authorization

Licence authorization communicates only the technical information
required to register, identify and validate the Lumen installation and
its authorization state.

This includes, as applicable:

-   the issued registration key during initial registration;
-   installation identity;
-   distribution and software version information;
-   runtime/authorization information; and
-   cryptographic authorization material.

Lumen does **not** transmit the researcher's:

-   prompts;
-   model responses;
-   conversations;
-   Trace contents;
-   Replay contents;
-   research results;
-   user documents;
-   datasets; or
-   other substantive research content

as part of licence authorization.

Researcher administrative information held by Illuminates.One is
separate from the technical authorization payload.

Transport-level information observed by Illuminates.One infrastructure
is an Illuminates.One privacy/operations matter and is not represented
as Lumen research-content telemetry.

------------------------------------------------------------------------

## 12. Code and Distribution Protection

M0.1 does not attempt to make locally executed software impossible to
inspect or modify.

Its practical protection model is instead based on several complementary
boundaries:

### 12.1 Authorization

An unmodified Lumen installation requires valid authorization from
Illuminates.One.

### 12.2 Cryptographic installation identity

Copying identifiers or registration data alone is not equivalent to
possessing a valid registered installation identity.

### 12.3 Signed authority

An operator cannot create a valid Illuminates.One authorization response
merely by possessing the distributed Lumen code or Illuminates.One
public verification material.

### 12.4 Controlled distribution

Official M0.1 research distributions are issued through Illuminates.One
and are associated with the intended research registration process.

### 12.5 Servire lifecycle control

Normal operation of the managed Stack is gated and controlled through
Servire.

### 12.6 Research Licence

The Research Licence provides the contractual boundary governing
permitted use, redistribution, reverse engineering, modification,
derivative/forked versions, commercial use and circumvention of licence
authorization.

Separate third-party licence rights continue to apply to third-party
components included with the distribution.

------------------------------------------------------------------------

## 13. Third-Party Components

Rogare uses third-party components that retain their own licence terms.

For the frozen M0.1 dependency set currently identified:

-   `@earendil-works/pi-coding-agent` 0.84.0 --- MIT --- Mario Zechner;
-   `pi-acp` 0.0.31 --- MIT --- Sergii Kozak.

Their licence notices are shipped separately in the Servire `licences/`
directory and are also acknowledged by the Lumen Research Licence.

Lumen's proprietary Research Licence restrictions must not be
represented as replacing rights granted by the applicable third-party
licences.

------------------------------------------------------------------------

## 14. What M0.1 Does Not Claim

M0.1 does not claim:

-   absolute tamper resistance;
-   that source or container contents cannot be inspected;
-   that an operator controlling the host cannot attempt to patch local
    software;
-   hardware-backed installation identity unless such protection is
    actually deployed;
-   prevention of every deliberate circumvention technique;
-   a complete future commercial licensing system; or
-   protection equivalent to a trusted hardware execution environment.

The objective is to make normal authorized operation clear and
controlled, prevent accidental or straightforward unauthorized reuse,
establish cryptographic provenance, and make deliberate circumvention
distinct from ordinary supported operation.

------------------------------------------------------------------------

## 15. Relationship to the Research Licence

Runtime authorization and the Research Licence serve different purposes.

**Runtime authorization is a technical control.**

**The Research Licence is a contractual control.**

The M0.1 Research Licence:

-   grants single-user research use;
-   prohibits commercial/production use;
-   prohibits redistribution of Lumen;
-   restricts reverse engineering and creation of Lumen forks/derivative
    versions;
-   prohibits deliberate defeat of the licence-authorization mechanism;
-   requires periodic licence authorization; and
-   leaves the researcher's permitted research and research results
    outside Lumen's ownership claim.

The licence intentionally does not expose internal authorization timing,
lease, retry or cryptographic implementation details.

------------------------------------------------------------------------

## 16. Release Evidence

The M0.1 authorization implementation has been completed and
live-proven.

Recorded acceptance evidence includes:

-   Mongo-backed researcher, registration, installation and
    authorization evidence;
-   persistent Ed25519 installation identity;
-   immutable installation/public-key binding after registration;
-   signed initial registration;
-   signed renewal through Licentia;
-   independent Servire verification of Illuminates.One decisions;
-   authorization lifecycle and recovery;
-   signed `LOCKED` state and recovery;
-   protected Stack authorization gating;
-   Servire authorization-state UI; and
-   successful registration and authorization from a clean M0.1
    installation.

The authorization implementation is therefore treated as **CLOSED for
M0.1**, subject to final release configuration checks and final
distribution-image acceptance.

------------------------------------------------------------------------

## 17. Final Release Checks --- CLOSED

The M0.1 release checks are closed for the frozen runtime baseline:

-   [x] release authorization timings confirmed: 24-hour lease, \~6-hour
    normal renewal, \~1-hour retry;
-   [x] fresh first registration and signed authorization proven from
    the Registry-distributed image;
-   [x] persistent installation identity/restart lifecycle proven during
    authorization acceptance;
-   [x] Stack authorization gating and Illuminates.One signature
    verification proven;
-   [x] clean re-pull with previous containers, volumes and cached Lumen
    image removed proved that installation identity and licence
    acceptance are not baked into the image;
-   [x] the frozen M0.1 Research Licence is presented on first start;
-   [x] Pi and `pi-acp` third-party licence notices are part of the
    defined release boundary;
-   [x] the exact frozen Registry image was used for final clean-install
    acceptance;
-   [x] the frozen image identity/digest is recorded in the
    release/distribution evidence.

Per-researcher Registry credentials and registration records remain
provisioning data and are not part of the reusable frozen runtime image.

------------------------------------------------------------------------

## 18. Superseded Document

This document **replaces and supersedes**:

> `M0.1_RUNTIME_AUTHORIZATION_AND_CODE_PROTECTION.md`

The superseded document should be removed rather than retained as an
active Lumen architecture document.

Any current M0.1 requirements or roadmap documents that reference the
superseded title should be updated to reference:

> **Lumen M0.1 Runtime Authorization and Code Protection --- Release
> Position**

The private authorization/evidence and Servire authorization-lifecycle
documents remain the implementation-level records for M0.1.

------------------------------------------------------------------------

## 19. M0.1 Release Position

The final M0.1 position is:

> **Illuminates.One authorizes the research installation. Servire owns
> the installation cryptographic identity, independently verifies
> Illuminates.One authorization, maintains the local authorization
> lifecycle and gates operation of the managed Lumen Stack. Licentia
> transports authorization exchanges without becoming an authorization
> authority. Periodic authorization provides a technical operating
> boundary, while the Lumen Research Licence provides the contractual
> use boundary. These controls provide practical release and provenance
> protection without claiming that software executing on
> researcher-controlled hardware is impossible to inspect, modify or
> circumvent.**
