# Lumen Commercial Development & Topology Requirements

**Status:** Early Architecture / Development Requirements  
**Purpose:** Preserve requirements emerging from consideration of future commercial Lumen deployment. These are not final commercial licensing decisions.

---

## 1. Context

The external Research distribution establishes a deliberately constrained deployment model in which the Lumen service stack operates on a single machine.

A future commercial Lumen deployment should remove that topology restriction and support Lumen as shared organisational infrastructure.

This introduces requirements beyond licensing.

In particular, Lumen must support:

- distributed services;
- multiple users;
- authenticated access;
- user isolation;
- permissions;
- shared infrastructure state;
- user-specific evidence;
- remote Servire instances; and
- administrative control.

These requirements represent significant future development and should not be allowed to delay the initial Research distribution.

---

## 2. Distributed Lumen

Commercial Lumen should permit individual Lumen services to execute on different machines.

For example:

```text
                 Lumen Deployment

        ┌────────────┐    ┌────────────┐
        │   Host A   │    │   Host B   │
        │ Moderari   │    │   Trace    │
        │ Pontis     │    │   Replay   │
        └──────┬─────┘    └─────┬──────┘
               │                │
               └───────┬────────┘
                       │
                ┌──────┴──────┐
                │   Host C    │
                │ Aestimare   │
                │ Praebere    │
                └─────────────┘
```

The particular topology must not be prescribed by Lumen.

A commercial deployment could remain entirely on one machine or be distributed according to operational requirements.

---

## 3. Servire as Remote Interface

Servire should not require installation on the machines hosting the Lumen services.

A user should be able to operate a local Servire instance connected to a remote Lumen deployment.

Conceptually:

```text
                     Shared Lumen

               ┌───────────────────┐
               │ Lumen Services    │
               │                   │
               │ Moderari          │
               │ Trace             │
               │ Replay            │
               │ Aestimare         │
               │ Praebere          │
               │ ...               │
               └─────────┬─────────┘
                         │
                  authenticated
                      access
                         │
              ┌──────────┴──────────┐
              │                     │
       User A Servire         User B Servire
       workstation            workstation
```

The Servire user experience should remain substantially consistent regardless of whether Lumen is local or remote.

---

## 4. Shared State vs User State

A commercial multi-user deployment introduces an important distinction between:

### Shared infrastructure state

Examples include:

- service running/stopped state;
- service health;
- service availability;
- deployed service versions;
- overall topology;
- infrastructure status.

Users may legitimately need visibility of this information.

### User-specific state

Examples include:

- sessions;
- prompts;
- responses;
- Trace evidence;
- Replay evidence;
- assessment results;
- user-specific operational events;
- other reasoning-assurance evidence.

Users should normally see only information belonging to their own identity and authorised scope.

---

## 5. Authentication

Commercial Lumen will require authenticated users.

A user identity must be capable of being propagated through the relevant Lumen services so that ownership and permissions can be enforced consistently.

Authentication architecture has not yet been selected.

It should not be assumed that Servire itself constitutes the authentication or security boundary.

---

## 6. User Isolation

User isolation must be enforced by the Lumen services holding or exposing user-specific information.

For example:

> Trace must return only Trace evidence that the authenticated identity is authorised to access.

Servire must not obtain unrestricted organisation-wide evidence and merely hide unauthorised records in its UI.

This principle applies to:

- sessions;
- Trace;
- Replay;
- Aestimare;
- operational events;
- logs containing user-specific information;
- provenance;
- future reasoning-assurance evidence.

**Filtering in Servire is presentation. Authorisation in the service is security.**

---

## 7. Servire Permissions

Servire will require a permission model.

The initial requirement need not be a complex enterprise RBAC implementation.

Two initial roles may be sufficient:

### User

A normal user may:

- view overall Lumen service state;
- view overall service health;
- use Lumen;
- view their own sessions;
- view their own prompts and responses;
- view their own Trace evidence;
- view their own Replay evidence;
- view their own Aestimare results;
- view operational information relating to their own activity.

A normal user may not:

- start or stop the Lumen stack;
- restart infrastructure services;
- change deployment-wide configuration;
- view another user's private evidence solely because it resides within the same Lumen deployment.

### Administrator

An administrator may additionally:

- start Lumen services;
- stop Lumen services;
- restart Lumen services;
- control the overall stack;
- view system-wide operational logs;
- administer deployment configuration;
- administer users and permissions where applicable.

Whether an administrator automatically has access to all users' prompts, traces and reasoning evidence should **not** be assumed.

Administrative infrastructure authority and authority to inspect user content may eventually need to be separate permissions.

---

## 8. Operational Logs

Servire currently exposes operational logging associated with Lumen services.

In a multi-user environment, operational logs may contain both:

- infrastructure-wide information; and
- information associated with individual user sessions.

Normal users should receive only operational information they are authorised to see.

Administrators may require broader infrastructure logging.

This filtering must ultimately be supported by structured identity/session metadata rather than relying upon textual filtering of log messages.

---

## 9. Session Ownership

Lumen will require an explicit concept of session ownership.

User identity should therefore become part of the provenance associated with relevant activity.

This potentially affects:

- Moderari;
- Pontis;
- Trace;
- Replay;
- Aestimare;
- Fiducia;
- Servire;
- persistent storage;
- operational event transport.

The precise propagation mechanism remains to be designed.

---

## 10. Commercial Topology Principle

The intended topology distinction is:

```text
RESEARCH

┌─────────────────────┐
│ One Machine         │
│                     │
│ Lumen Services      │
│ Servire             │
└─────────────────────┘
```

and:

```text
COMMERCIAL

      User Servire     User Servire
           │                │
           └───────┬────────┘
                   │
             Lumen Deployment
                   │
        ┌──────────┼──────────┐
        │          │          │
      Host A     Host B     Host C
```

A commercial Lumen deployment may therefore exist independently of the machines from which individual users interact with it.

---

## 11. Licensing Implications

No final commercial licensing model is defined by this document.

However, this topology makes a per-user commercial model technically meaningful.

A commercial Lumen user would represent an authenticated identity authorised to use a shared Lumen reasoning-assurance deployment rather than necessarily possessing a complete local installation of Lumen.

A future licensing hierarchy could therefore potentially represent:

```text
Organisation
    │
    └── Lumen Deployment
            │
            ├── User
            ├── User
            └── User
```

Licensed capabilities, including future Aestimare Assessors, could potentially be attached at different levels within that hierarchy.

Those commercial decisions remain intentionally open.

---

## 12. Aestimare

Commercial Lumen is presently expected to include Aestimare.

Aestimare is expected to provide a core collection of assessment capabilities with additional specialist assessment capabilities potentially licensed separately.

The exact commercial packaging, included Assessors, pricing and licensing scope remain undecided.

They should be determined as Aestimare's architecture and capabilities mature.

---

## 13. Development Implications

Supporting this topology will require substantial development, potentially including:

- authentication;
- user identity;
- authorisation;
- role/permission handling;
- identity propagation between services;
- session ownership;
- evidence ownership;
- database/schema changes;
- structured operational events;
- remote Servire connectivity;
- secure service-to-service communication;
- distributed service discovery/configuration;
- administrator controls;
- user management;
- licence identity and entitlement handling.

These requirements should be developed incrementally rather than introduced as one large commercialisation milestone.

---

## 14. Architectural Principle

The key architectural principle arising from the commercial topology is:

> **Lumen is shared reasoning-assurance infrastructure; Servire is an authenticated view onto that infrastructure appropriate to the identity and authority of its user.**

This allows the same fundamental Lumen architecture to progress from:

**single-machine Research deployment → commercial deployment → shared organisational infrastructure → distributed Lumen infrastructure**

without requiring separate products for each deployment model.

---

## 15. Status

This document records requirements and architectural consequences identified while considering future commercial deployment.

It does **not** establish:

- commercial pricing;
- final licensing units;
- subscription terms;
- included commercial capabilities;
- Assessor pricing;
- enterprise licensing;
- support commitments; or
- a commercial release schedule.

Those decisions should remain open until the Research licensing and distribution model has been established and practical experience with external Lumen use has been obtained.