# Lumen Aestimare

Lumen Aestimare is Lumen's assessment service. It evaluates evidence about model behaviour and contributes specialised assessment to the wider Reasoning Assurance architecture.

Aestimare is not intended to operate as a single monolithic assessment algorithm. Assessment is decomposed into specialist domains represented by **Periti Assessores** (Expert Assessors). Each **Peritus** examines a distinct assessment question using evidence and methodology appropriate to that domain, while Aestimare coordinates their findings into a broader assessment.

Periti should emerge from identified assessment problems rather than from a predetermined taxonomy. A new Peritus exists because there is a distinct question requiring specialised evidence and methodology, not simply because the architecture has room for another assessor.


## Assessment Principle

> **Assess measures behaviour, not intelligence.**

Aestimare treats model executions as observable performances. It seeks to characterise how trained models perform under particular tasks and conditions, how consistently that performance can be reproduced, what changes that performance, and what evidence accumulates over time.

Its concern is **fitness for purpose**, rather than establishing a universal ranking of model intelligence.


## Periti Assessores

Specialist assessor documentation is maintained under [`periti_assessores/`](periti_assessores/).

The first defined Peritus is:

- **Aestimare Oeconomia — Economic Efficiency Assessor** — examines the relationship between satisfactory task outcomes and the total economic cost of obtaining and consuming those outcomes, including computational cost and human interaction burden.

See [`periti_assessores/README.md`](periti_assessores/README.md) for the assessor architecture and naming convention.

## Documents

- [AESTIMARE_MODEL_PERFORMANCE_AS_AN_EXPERIMENTAL_DISCIPLINE.md](AESTIMARE_MODEL_PERFORMANCE_AS_AN_EXPERIMENTAL_DISCIPLINE.md)
- [AESTIMARE_BEHAVIOURAL_STABILITY_EXPERIMENTAL_METHODOLOGY.md](AESTIMARE_BEHAVIOURAL_STABILITY_EXPERIMENTAL_METHODOLOGY.md)
- [AESTIMARE_EXTERNAL_EVIDENCE_AND_COMPLEMENTARY_CAPABILITIES.md](AESTIMARE_EXTERNAL_EVIDENCE_AND_COMPLEMENTARY_CAPABILITIES.md)
- [LUMEN_ASSESS_BEHAVIOURAL_EVALUATION.md](LUMEN_ASSESS_BEHAVIOURAL_EVALUATION.md)
- [LUMEN_ASSESS_OVERVIEW.md](LUMEN_ASSESS_OVERVIEW.md)
- [LUMEN_ASSESS_REFINED_ROLE_REASONING_ASSURANCE_2026-08-10.md](LUMEN_ASSESS_REFINED_ROLE_REASONING_ASSURANCE_2026-08-10.md)
- [Lumen-Assess-Initial-Development-Corpus.md](Lumen-Assess-Initial-Development-Corpus.md)
- [WE_CRITICISE_AI_FOR_BEHAVING_LIKE_US.md](WE_CRITICISE_AI_FOR_BEHAVING_LIKE_US.md)
