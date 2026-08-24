# Lumen Commercial Capability-Based Pricing Concept

**Status:** Exploratory Concept  
**Purpose:** Preserve an emerging commercial pricing idea for future consideration.

> This document records a pricing concept, not a pricing decision. It deliberately does not define prices, implementation mechanisms, migration processes or final licensing terms.

---

## 1. Background

Lumen is composed of independent services rather than being a single monolithic application.

Although the complete Lumen ecosystem contains a growing collection of services, not every Lumen use case necessarily requires every service.

A deployment concerned primarily with context management and continuity, for example, may require only a relatively small core collection of Lumen services.

A deployment concerned with deeper reasoning assurance may additionally require capabilities such as Trace, Replay, Aestimare and other Lumen services.

This raises the possibility that commercial pricing should reflect the **capabilities actually required by a deployment**, rather than requiring every commercial user to license the complete Lumen ecosystem.

---

## 2. Lumen Core

A minimum collection of services could form a commercial **Lumen Core**.

The current architectural thinking suggests that such a core may include services such as:

- Pontis;
- Praebere;
- Nuntius; and
- Moderari.

This list is illustrative and reflects the architecture at the time this concept was recorded.

It should not be interpreted as a permanent definition of Lumen Core.

The purpose of the Core concept is to establish a useful minimum Lumen deployment upon which additional capabilities can be composed.

---

## 3. Capability-Based Commercial Model

Rather than licensing the complete Lumen service family as one indivisible package, commercial Lumen could potentially be licensed according to the services required by the customer.

Conceptually:

```text
Lumen Core
    │
    ├── Core Service
    ├── Core Service
    ├── Core Service
    └── Core Service
            │
            ├── + Trace
            ├── + Replay
            ├── + Aestimare
            ├── + Fiducia
            └── + other Lumen capabilities
```

The commercial cost would therefore grow as additional Lumen capabilities are required.

This would make the commercial model reflect the compositional architecture of Lumen itself.

---

## 4. Incremental Adoption

A capability-based model would allow an organisation to begin with Lumen Core and extend its Lumen capabilities later.

For example, an organisation might initially require only context and continuity capabilities.

Later it might determine that it also requires:

- behavioural evidence through Trace;
- reproduction through Replay;
- assessment through Aestimare; or
- another future Lumen capability.

The organisation could then subscribe to the additional capability rather than replacing its existing Lumen subscription with an artificial product tier.

Conceptually:

```text
Lumen Core
     │
     ▼
Lumen Core + Trace
     │
     ▼
Lumen Core + Trace + Replay
     │
     ▼
Lumen Core + Trace + Replay + Aestimare
```

There is no requirement that organisations follow this sequence.

The principle is simply that Lumen could grow with the organisation's requirements.

---

## 5. Service-Based Pricing

One possible pricing principle is that individual Lumen services have a small recurring per-user value.

A commercial deployment would therefore consist of:

> **minimum Lumen Core subscription + additional subscribed Lumen services**

The exact monetary value is deliberately not specified in this document.

The attraction of the model is its simplicity.

A customer requiring a relatively small Lumen capability set pays for a relatively small Lumen capability set.

A customer making broader use of the reasoning-assurance ecosystem subscribes to more of that ecosystem.

---

## 6. Dependencies

Lumen services are not necessarily independent from an architectural perspective.

Some capabilities require other Lumen services to operate.

Capability-based pricing therefore does not imply that every possible combination of services must be commercially or technically valid.

For example:

```text
Requested Capability
        │
        ▼
Required Lumen Services
        │
        ▼
Valid Lumen Deployment
```

The commercial model should respect Lumen's architectural dependency relationships rather than creating artificial combinations purely for pricing purposes.

---

## 7. Aestimare

Aestimare introduces a further dimension to the capability-based model.

The Aestimare service itself could form one licensable Lumen capability while providing a collection of generally applicable Core assessment capabilities.

Additional specialist assessment capabilities — Periti — could then be separately available.

Conceptually:

```text
Aestimare
    │
    ├── Core assessment capabilities
    │
    ├── Peritus
    ├── Peritus
    ├── Peritus
    └── ...
```

This reflects the intended architecture in which individual Periti provide specialised assessment expertise rather than Aestimare operating as one monolithic assessment algorithm.

---

## 8. Specialist Periti

Specialist Periti may not all have equivalent commercial value.

Their value could depend upon factors such as:

- assessment sophistication;
- specialisation;
- domain expertise;
- computational requirements;
- development and maintenance effort;
- applicability; and
- value of the evidence they produce.

Consequently, a uniform price for every Peritus may not be appropriate.

A future commercial model could instead allow specialist Periti to occupy different pricing levels according to their capability and expertise.

No pricing scale is proposed by this document.

---

## 9. Composable Commercial Lumen

The broader idea can therefore be expressed as:

```text
                    Lumen Core
                        │
        ┌───────────────┼────────────────┐
        │               │                │
     Service         Service          Service
        │
     Aestimare
        │
   ┌────┼────┬────────────┐
   │    │    │            │
 Core  Peritus         Peritus ...
```

Commercial Lumen would become economically composable in much the same way that it is architecturally composable.

Customers would not necessarily purchase a predefined edition containing capabilities they do not require.

Instead, they could construct the Lumen capability set appropriate to their use.

---

## 10. Why This May Be Attractive

This concept has several potentially useful characteristics.

### Low barrier to entry

An organisation with a relatively modest Lumen requirement would not need to subscribe to the complete ecosystem.

### Incremental adoption

Lumen capabilities could be added as the organisation discovers additional requirements.

### Architectural alignment

Commercial packaging would reflect Lumen's service architecture rather than imposing unrelated artificial product tiers upon it.

### Transparent value

Additional cost corresponds to additional capability.

### Extensibility

Future Lumen services could potentially enter the same commercial structure without requiring the entire pricing model to be redesigned.

### Aestimare specialisation

Specialist Periti can develop independently and potentially carry commercial value appropriate to their particular expertise.

---

## 11. Questions for the Future

This concept intentionally leaves substantial questions unanswered.

Among them are:

- What ultimately constitutes Lumen Core?
- Is per-user pricing the appropriate unit?
- Should every additional Lumen service have the same commercial value?
- Should some services be included automatically because they are dependencies?
- Should certain capabilities be licensed per organisation rather than per user?
- How should specialist Periti be valued?
- Should Core Aestimare capabilities be included with Aestimare?
- At what point does individually composed pricing become unnecessarily complicated?
- Would customers prefer composition or a small number of predefined bundles?
- How should very large deployments be treated?
- How does this model interact with enterprise licensing?
- Does practical experience with Lumen support the assumptions behind this concept at all?

These are deliberately questions rather than decisions.

---

## 12. Status of the Concept

No commitment has been made to use capability-based commercial pricing.

No prices have been established.

No implementation or migration mechanism is defined by this document.

No existing Commercial licensing position is superseded by it.

This document exists because the idea appears sufficiently aligned with Lumen's architecture to be worth preserving.

When commercial licensing becomes an active requirement, this concept should be reconsidered alongside:

- the architecture Lumen has actually developed;
- experience from Research distribution;
- actual customer usage;
- the eventual role of Aestimare and its Periti;
- operational costs;
- commercial feedback; and
- other pricing models identified in the intervening period.

The conclusion at that time may be to adopt this model, modify it substantially, combine it with another model or discard it entirely.

Its purpose now is simply to ensure that the reasoning behind the idea is available when that decision eventually needs to be made.