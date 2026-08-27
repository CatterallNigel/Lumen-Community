# Lumen Commercial Licensing — Working Position

**Status:** Early Research / Working Position  
**Purpose:** Preserve the current direction and principles being considered for future commercial Lumen licensing.

> This document does not define final commercial pricing, licence terms or contractual commitments. It records the current working position so that future commercial licensing decisions can continue from an established foundation.

---

## Version History

| Version | Date | Status | Changes |
|---|---|---|---|
| 0.1 | 2026-08-22 | Initial working draft | Initial document. |
| 0.2 | 2026-08-26 | Working draft | Added commercial value progression for Lumen evidence, Aestimare assurance, and specialist Periti. |

---

## 1. Purpose

Lumen is expected to have separate Research and Commercial licensing models.

The Research licence will be established first and will provide practical experience with:

- external distribution;
- registration;
- licence identity;
- heartbeat validation;
- updates;
- installation management; and
- external use of Lumen.

The Commercial licence will build upon those mechanisms while enabling capabilities and deployment models deliberately excluded from the Research distribution.

Commercial licensing does not need to be finalised before the Research distribution is released.

The objective at this stage is to preserve the principles already identified without prematurely fixing implementation or pricing decisions.

---

## 2. Commercial Use

A Commercial licence will permit Lumen to be used for commercial and production purposes.

This is expected to include:

- internal organisational use;
- production deployment;
- commercial development;
- use of Lumen in support of commercial products and services; and
- distributed Lumen infrastructure.

The eventual licence will need to define the boundaries of permitted commercial use more precisely.

---

## 3. Commercial Deployment

Unlike the Research licence, the Commercial licence is expected to permit Lumen services to be distributed across multiple machines.

A commercial Lumen deployment may therefore range from:

```text
Single Machine
     │
     └── Lumen
```

through to:

```text
              Lumen Deployment
                     │
        ┌────────────┼────────────┐
        │            │            │
      Host A       Host B       Host C
```

The licence should not require a distributed topology.

It should simply permit one.

This reflects an architectural capability of commercial Lumen rather than an arbitrary licensing distinction.

---

## 4. Multi-User Commercial Lumen

Commercial Lumen is expected eventually to support multiple users sharing common Lumen infrastructure.

Users may operate Servire independently of the machines hosting the Lumen services.

For example:

```text
      User Servire      User Servire
            │                 │
            └────────┬────────┘
                     │
              Lumen Deployment
                     │
          ┌──────────┼──────────┐
          │          │          │
        Host A     Host B     Host C
```

Each user should have access to the Lumen capabilities for which they are authorised while user-specific information remains appropriately isolated.

The detailed architecture for authentication, authorisation and user isolation is documented separately as a commercial development requirement.

---

## 5. Subscription Model

Commercial Lumen is presently expected to use a **subscription licensing model**.

The current working structure is:

**Trial → Initial Subscription Period → Monthly Subscription**

A prospective commercial user would initially receive a free evaluation period.

Following that trial, continued use would require an initial minimum subscription period.

After completion of the initial subscription period, the subscription would continue monthly and could be cancelled without requiring another long-term commitment.

The present working assumption is approximately:

- a **30-day trial**;
- an initial minimum subscription period of approximately **three months**; and
- monthly subscription thereafter.

These periods are indicative rather than final commitments.

They should be reviewed before commercial release.

---

## 6. Pricing Direction

Commercial Lumen is intended to have a deliberately accessible entry price.

The present direction is toward a relatively low monthly base cost associated with authorised commercial use.

A **per-user monthly model** is currently considered a plausible approach.

However, no commercial price should presently be treated as final or published as a commitment.

The eventual price should be determined after:

- Research distribution experience;
- development of commercial multi-user capabilities;
- Aestimare development;
- understanding actual operational costs;
- external feedback;
- early commercial evaluation; and
- clearer understanding of the value Lumen provides in production environments.

The current pricing discussions should therefore be regarded as **order-of-magnitude planning assumptions only**.

---

## 7. Licensing Unit

The precise commercial licensing unit remains unresolved.

Potential concepts include:

- named users;
- concurrent users;
- installations;
- deployments;
- organisational entitlements; or
- combinations of these.

The current architecture suggests that several identities may need to remain conceptually separate:

```text
Commercial Licence
        │
        ├── Deployment
        │       ├── User
        │       └── User
        │
        └── Deployment
                └── User
```

The existence of these identities does not determine which of them becomes the billable licensing unit.

That decision should remain open until the commercial architecture and expected usage patterns are better understood.

---

## 8. Licence and Deployment Identity

The commercial licensing mechanism should distinguish between:

### Licence Entitlement

Identifies the commercial subscription and the capabilities it permits.

### Deployment Identity

Identifies a particular authorised Lumen deployment or installation associated with that entitlement.

### User Identity

Identifies an authorised person using the commercial Lumen infrastructure.

These identities may interact with licensing, but they should not be assumed to be interchangeable.

This distinction allows the eventual licensing model to evolve without requiring fundamental changes to Lumen's licensing architecture.

---

## 9. Heartbeat and Licence Validation

Commercial Lumen is expected to use the same fundamental heartbeat principle established for the Research distribution.

The heartbeat should disclose the **minimum information necessary** to validate the licence.

It should not transmit substantive Lumen activity such as:

- prompts;
- responses;
- Trace evidence;
- Replay evidence;
- Aestimare results;
- documents;
- research or business data; or
- model-generated content.

The commercial heartbeat may identify the licence entitlement and deployment sufficiently for Illuminates.One to determine whether the deployment remains authorised.

Conceptually:

```text
Lumen
  │
  │ Licence identity
  │ Deployment identity
  ▼
Illuminates.One
  │
  ├── Licence active?
  ├── Deployment authorised?
  ├── Capacity available?
  ├── Capabilities permitted?
  └── Subscription valid?
```

The precise protocol remains to be designed.

---

## 10. User Capacity

A future Commercial licence is expected to establish some form of permitted user capacity.

The mechanism by which that capacity is measured remains deliberately unresolved.

It may ultimately involve:

- registered users;
- named users;
- concurrently active users;
- authorised Servire installations;
- deployment registrations; or
- another mechanism.

The important current principle is:

> **The licensing architecture should be capable of validating authorised commercial capacity without requiring Illuminates.One to receive users' substantive Lumen activity.**

The Research licensing implementation should avoid architectural decisions that unnecessarily prevent this future extension.

---

## 11. Aestimare

Commercial Lumen is expected to include **Lumen Aestimare**.

The intended direction is for the Commercial base subscription to include a useful collection of **Core Assessors**.

Additional specialist Assessors may be separately licensed.

Conceptually:

```text
Commercial Lumen
      │
      └── Aestimare
             │
             ├── Core Assessors
             │      Included
             │
             ├── Specialist Assessor
             │      Optional
             │
             ├── Specialist Assessor
             │      Optional
             │
             └── ...
```

Additional Assessors are presently expected to use a recurring subscription model.

Pricing may differ according to the capability, complexity, specialisation or value of an Assessor.

No Assessor pricing is presently final.

---


## 11.1 Commercial Value Progression

The developing capability-based pricing work suggests a useful distinction between the commercial roles of the wider Lumen architecture, Aestimare and specialist Periti:

> **Lumen creates and preserves the evidence. Aestimare makes that evidence valuable. Periti determine how specialised that value can become.**

The statement should not be interpreted as implying that the core Lumen services have little value. Those services establish the continuity, provenance, reproduction and experimental evidence upon which meaningful assessment depends.

Rather, it identifies a possible progression of commercial value:

```text
Lumen evidence substrate
        |
        v
Aestimare assurance and interpretation
        |
        v
Specialist Periti expertise
```

This may ultimately justify a commercial structure in which access to core Lumen capabilities remains deliberately accessible while Aestimare and specialised Periti carry value appropriate to the assurance and domain expertise they provide.

Market-landscape research also indicates that evaluator extensibility exists in several adjacent AI observability and evaluation systems. The commercial proposition should therefore not depend on the claim that extensible evaluation is unique to Lumen. The question to preserve for later validation is whether Lumen's combination of accumulated evidence, continuity, provenance, controlled reproduction, longitudinal behavioural characterisation and independently specialised Periti creates a meaningfully different assurance model.

The answer should be established through architecture, research evidence, external use and market comparison before becoming a formal commercial claim.


## 12. Assessor Updates

While an Assessor subscription remains active, updates to that licensed Assessor are expected to be included.

Customers should not ordinarily need to purchase a new licence merely because an existing licensed Assessor has been improved or updated.

This principle does not prevent substantially different future assessment capabilities from being introduced as separate Assessors.

---

## 13. Commercial Updates

An active Commercial subscription is expected to include normal updates to the Lumen capabilities covered by that subscription.

This establishes the general principle:

> **An active subscription provides continued entitlement to supported updates for the capabilities being licensed.**

The eventual commercial release policy will need to distinguish between:

- updates;
- new capabilities;
- optional components;
- specialist Assessors; and
- potentially major future commercial offerings.

---

## 14. Relationship with Research Licensing

The Research licence should provide the initial foundation for the commercial licensing mechanism.

In particular, Research will provide practical experience with:

```text
Registration
      │
      ▼
Entitlement
      │
      ▼
Distribution Identity
      │
      ▼
Deployment
      │
      ▼
Heartbeat
      │
      ▼
Licence Validation
```

Commercial licensing can extend this model with:

- commercial entitlements;
- distributed deployment;
- multiple users;
- subscription validation;
- licensed capacity;
- Aestimare;
- optional Assessors; and
- additional commercial capabilities.

The Commercial licence should therefore evolve from the Research licensing architecture rather than becoming an unrelated licensing system.

---

## 15. Current Commercial Proposition

At the present stage, the intended commercial proposition can be summarised as:

**Commercial and production use — subscription licensing — initial free trial — short initial minimum commitment — monthly continuation — distributed deployment permitted — multi-user capability — Aestimare Core included — optional specialist Assessors — updates included while subscribed — mandatory privacy-minimising licence heartbeat.**

This is a **working proposition**, not a final product offering.

---

## 16. Decisions Deliberately Left Open

The following should not be finalised prematurely:

- Final subscription price.
- Exact trial duration.
- Exact initial minimum subscription period.
- Per-user versus alternative licensing units.
- Named versus concurrent users.
- Relationship between users and deployments.
- Number of deployments permitted by a subscription.
- Treatment of development, test and production deployments.
- Organisation-level licensing.
- Enterprise licensing.
- Volume pricing.
- Educational or non-profit commercial arrangements.
- Offline or air-gapped commercial licensing.
- Exact Core Assessors included with Aestimare.
- Specialist Assessor pricing.
- Whether particular Assessors are licensed per user, deployment or organisation.
- Support commitments.
- Service-level commitments.
- Commercial update policy.
- Subscription expiry behaviour.
- Grace periods.
- Licence transfer.
- Deployment replacement and recovery.
- Commercial heartbeat protocol.
- Commercial heartbeat retention policy.
- Jurisdiction and governing law.

These matters should be resolved when there is sufficient evidence to make the decisions rather than simply because a commercial licensing document exists.

---

## 17. Guiding Principle

Commercial licensing should remain understandable, proportionate and closely related to the capabilities Lumen actually provides.

The objective is not to construct a complicated licensing system.

The objective is to establish a sustainable commercial model in which:

> **Research users can genuinely investigate Lumen, commercial users can deploy and operate it without artificial architectural restrictions, and additional specialised reasoning-assurance capabilities can evolve independently as the Lumen ecosystem develops.**

This document should be revisited after practical experience has been obtained from the external Research distribution and before formal Commercial licence terms are prepared.