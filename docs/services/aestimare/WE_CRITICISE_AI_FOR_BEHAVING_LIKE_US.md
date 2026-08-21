# We Criticise AI for Behaving Like Us

## What behavioural science can teach Reasoning Assurance about probabilistic systems

**Lumen Research / Aestimare**\
**Illuminate.One**\
**August 2026**

------------------------------------------------------------------------

## Abstract

We often judge artificial intelligence against an ideal of rational,
consistent behaviour that human beings themselves do not exhibit.

Humans are influenced by framing and context. We exhibit systematic
biases. Memory is reconstructive. We rationalise decisions after making
them. We satisfice rather than optimise. Different people presented with
apparently similar evidence can reach different conclusions, and the
same person can reach different conclusions under changed conditions.

When large language models exhibit analogous *observable phenomena*,
however, we frequently treat them as uniquely alarming properties of
artificial intelligence: bias, inconsistency, context sensitivity,
confident error, divergent reasoning and behavioural drift.

This paper argues that deterministic software engineering may therefore
provide an incomplete baseline for evaluating probabilistic AI
behaviour. Behavioural psychology and cognitive science have spent
generations developing empirical methods for studying complex cognitive
systems without complete access to their internal mechanisms. Those
methods offer a potentially valuable foundation for Reasoning Assurance.

This is not an argument that an LLM has a human psychology, nor that
artificial and biological cognition are equivalent. The proposal is
methodological:

> **Borrow the methodology, not the ontology.**

Recent research strengthens the case for this approach. Tremblay et
al. (2026), in a narrative review drawing on 363 articles directly
contributing to a comparison of human and artificial cognition, identify
parallels in statistical processing, associative pattern recognition,
bias, memory distortion and decision-making opacity, while also
emphasising important differences in embodiment, intentionality and
phenomenology. Hagendorff et al. propose a "machine psychology" that
uses behavioural experiments inspired by psychology to investigate LLM
capabilities and behavioural patterns beyond conventional benchmarks.
Mitchell (2026) similarly argues that developmental and comparative
psychology provide methodological principles for more rigorous AI
evaluation.

Lumen extends this direction into an operational engineering problem. It
asks not merely what AI behaviour tells us about machine cognition, but:

> **What does accumulated behavioural evidence justify us relying upon
> this system to do?**

Reasoning Assurance therefore treats AI behaviour as observable,
reproducible, comparable, assessable and longitudinal. The objective is
not deterministic certainty. It is justified, revisable confidence
within known conditions and consequences.

------------------------------------------------------------------------

## 1. We Criticise AI for Behaving Like Us

We complain that AI is biased.

**Humans are biased.**

We complain that AI is inconsistent.

**Humans are inconsistent.**

We complain that context changes an AI's answer.

**Context changes human judgement.**

We complain that AI can confidently provide an incorrect explanation.

**Humans rationalise, misremember and confabulate.**

We complain that two instances given the same evidence can reach
different conclusions.

**Put twelve humans on a jury.**

None of this means that AI error is harmless, that AI and human
cognition are equivalent, or that undesirable model behaviour should
simply be accepted.

Quite the opposite.

It suggests that we may be applying the wrong expectation.

Software engineering has trained us to value reproducibility. For a
deterministic system, the same state and the same input should produce
the same result. Divergence is therefore informative. It can indicate a
defect, changed state, unexpected input, corrupted data or a broken
invariant.

Probabilistic systems complicate that assumption.

Two executions may follow different reasoning paths and still reach
equally acceptable outcomes. A model may choose different tools,
formulate a different explanation, or emphasise different evidence
without either execution being materially worse.

Variation alone is therefore not failure.

The engineering problem becomes distinguishing **acceptable behavioural
variation** from **material behavioural change**.

This leads to the central proposition of this paper:

> **We judge AI against an ideal of rational, consistent behaviour that
> humans themselves do not exhibit.**

If probabilistic behaviour is intrinsic to the system, demanding
deterministic reproduction may tell us less than characterising the
range within which that behaviour remains acceptable.

------------------------------------------------------------------------

## 2. The Wrong Baseline

Traditional software assurance begins with an enormous advantage:
engineers construct the mechanism.

We can inspect source code, define inputs, establish expected outputs,
construct unit and integration tests, reproduce faults, instrument
execution and alter the implementation when behaviour is wrong.

Even when modern distributed systems become too complex for any one
engineer to understand completely, the underlying expectation remains
largely deterministic. Observability helps us reconstruct what happened
across a complex execution.

Generative AI changes the nature of the problem.

We generally do not control the model's internal learned
representations. Even where model weights are available, inspecting
those weights does not provide an operational explanation equivalent to
reading conventional source code.

Instead, we influence behaviour through conditions around the model:

-   system instructions;
-   assistant and user messages;
-   accumulated context;
-   retrieved information;
-   available tools;
-   orchestration policy;
-   model and provider selection;
-   sampling and inference parameters;
-   prior interactions and state.

A prompt is therefore not equivalent to a conventional program.

It is better understood operationally as **an influence on a behavioural
distribution**.

Change the context and behaviour may change. Change the available tools
and the reasoning path may change. Substitute the model and a previously
successful instruction may produce a different result. Repeat apparently
identical conditions and the execution may still vary.

The traditional question:

> Did the system produce the expected output?

remains useful for tasks with definitive answers, but it is insufficient
for many reasoning tasks.

A more appropriate set of questions is:

-   What range of behaviour do we observe?
-   Which variations remain acceptable?
-   Which differences are material?
-   Under what conditions does behaviour change?
-   Are those changes reproducible?
-   Is the system drifting relative to previously observed behaviour?
-   What evidence is sufficient to justify reliance?

That is not weaker testing.

It is testing a different kind of system.

------------------------------------------------------------------------

## 3. Human Cognition Was Already This Problem

Psychology and cognitive science have long faced a related
methodological problem.

The human brain is observable physically, but cognition cannot simply be
inspected as source code. Researchers infer capabilities, tendencies and
mechanisms from behaviour under controlled and naturalistic conditions.

They manipulate stimuli. They construct control conditions. They repeat
experiments. They compare groups and individuals. They measure error.
They examine context effects. They identify biases. They form hypotheses
and attempt to falsify them.

In other words, behavioural science has developed a substantial
methodological toolbox for investigating complex systems through
observable behaviour.

Tremblay, Marois, Zare and Lafond's 2026 review, *Shared Minds: The
Cognitive Parallels Between Humans and Artificial Intelligence*,
provides an important contemporary foundation for this discussion. Their
final narrative corpus contained 370 publications, of which 363 directly
contributed to the comparative analysis. They compare human and
artificial cognition across learning, pattern recognition, perception,
cognitive load, memory, reasoning, problem solving and decision-making.

Their review argues against an idealised model of human rationality.
Human and artificial cognition both make extensive use of statistical
processing, associative pattern recognition and approximation. The
authors identify shared vulnerabilities including biases, memory
distortions and decision-making opacity, while explicitly retaining
important differences between biological and artificial systems.

This does not establish that LLMs think like humans.

It establishes something more useful for Reasoning Assurance: **human
rationality is itself an inappropriate deterministic benchmark**.

The historical connection is also important. Tremblay et al. trace the
information-processing tradition through cognitive psychology and
through Newell and Simon's work on human problem solving. Herbert Simon
argued that human and artificial intelligence could be examined through
common information-processing principles.

This is not a new bridge between psychology and computing.

What is new is the operational importance of that bridge as
probabilistic AI systems acquire responsibility inside production
systems.

------------------------------------------------------------------------

## 4. Borrow the Methodology, Not the Ontology

The comparison between human and artificial behaviour creates an obvious
danger: anthropomorphism.

An LLM producing a human-like response does not establish that the
internal process producing that response is equivalent to human
cognition.

We should therefore resist statements such as:

-   the model *believes* something;
-   the model *wants* something;
-   the model *knows* it is breaking a rule;
-   the model *thinks* it can get away with an action.

These descriptions may be conversationally convenient, but they
introduce claims about internal states that behavioural evidence alone
cannot establish.

Mitchell's 2026 work on evaluating cognitive capabilities in AI models
makes this warning explicit. Drawing on developmental and comparative
psychology, she argues that evaluators must be aware of their own
anthropomorphic biases and design experiments capable of distinguishing
alternative explanations for observed performance.

For Reasoning Assurance, the solution is simple:

> **Borrow the methodology, not the ontology.**

We do not need to establish whether an AI *thinks*, *believes*,
*understands* or possesses anything resembling a human mind.

We can make a much narrower statement:

> **It behaves.**

Behaviour can be observed.

Behaviour can be reproduced under controlled conditions.

Behaviour can be compared.

Behaviour can be characterised.

Behaviour can be assessed longitudinally.

Behaviour can be tested when conditions change.

Behaviour can be evaluated against operational boundaries.

That is enough to begin building evidence.

------------------------------------------------------------------------

## 5. From Psychology to Machine Psychology

The application of behavioural psychology to artificial intelligence is
not unexplored.

Hagendorff et al.'s *Machine Psychology* explicitly proposes engaging
LLMs in behavioural experiments inspired by psychology. Their objective
is to move beyond performance benchmarks toward investigation of
emergent abilities, behavioural patterns and computational mechanisms.

They also emphasise an important caution: methodologies developed for
humans cannot simply be transferred uncritically to machines.

Mitchell independently reaches a closely related position. Her six
principles for evaluating AI cognitive capabilities include:

1.  recognising anthropomorphic bias;
2.  designing control experiments for alternative explanations;
3.  creating novel variations to test robustness and generalisation;
4.  investigating mechanisms underlying performance;
5.  distinguishing performance from competence;
6.  analysing failure types and embracing negative results.

These are strikingly relevant to Reasoning Assurance.

A benchmark asks:

> How well did the model perform on this test?

A behavioural experiment asks:

> What does changing this condition tell us about the behaviour we
> observed?

Those are different questions.

Benchmarking remains valuable. But a single aggregate score can conceal
the behavioural structure beneath it: which tasks failed, how failures
cluster, whether small changes produce large effects, whether success
generalises, and whether apparently strong performance results from the
capability we believe is being measured.

Reasoning Assurance therefore has more in common with **experimental
investigation** than with a leaderboard.

------------------------------------------------------------------------

## 6. The Missing Operational Layer

Machine psychology primarily asks what behavioural experiments can teach
us about artificial cognition, capabilities and emergent behaviour.

Reasoning Assurance asks an additional operational question:

> **What can behavioural evidence justify us relying upon this deployed
> system to do?**

That distinction matters.

A model may demonstrate a capability in a benchmark without providing
sufficient evidence that an organisation should delegate a consequential
task to it.

Likewise, a model may perform inconsistently in superficial ways while
remaining entirely reliable with respect to the outcome that matters.

The object of assurance is therefore not the model in isolation.

It is the relationship between:

**system + task + conditions + consequences + evidence**

A model might be sufficiently reliable for summarising internal meeting
notes but insufficiently assured for deciding whether a financial
transaction should proceed.

The same model may deserve different levels of trust for different
tasks.

Trust therefore becomes contextual rather than intrinsic.

The useful statement is not:

> This model is trustworthy.

It is:

> Under these conditions, for this class of task and these consequences,
> accumulated evidence currently justifies this degree of reliance.

That confidence must remain revisable when the evidence changes.

------------------------------------------------------------------------

## 7. Behaviour as Evidence

If behavioural science is to inform Reasoning Assurance, the objective
is not to create a psychological personality profile for a model.

The objective is to identify behavioural dimensions that can be
observed, tested and assessed.

### 7.1 Bias

Humans exhibit systematic cognitive biases. AI systems also exhibit
measurable biases, although their origins and mechanisms need not be the
same.

The assurance question is operational:

-   Under what conditions does a bias appear?
-   Is it stable?
-   Can it be reproduced?
-   Does framing strengthen or weaken it?
-   Does it materially affect the task outcome?

### 7.2 Framing and context sensitivity

Human judgement can change when logically equivalent information is
framed differently.

LLM behaviour is also highly sensitive to presentation, instruction and
accumulated context.

This makes stimulus variation particularly relevant. An assurance corpus
should not merely repeat identical prompts. It should include controlled
variations that ought not materially alter a robust result.

### 7.3 Memory, reconstruction and interference

Human memory is not a perfect recording. It is reconstructive and
susceptible to interference.

LLM context is not human memory, but operationally similar questions can
be asked:

-   Does earlier information continue to influence later reasoning
    appropriately?
-   Does newer context overwrite or distort previously established
    information?
-   Does irrelevant context alter behaviour?
-   Does accumulated context produce drift?

The similarity is methodological, not mechanistic.

### 7.4 Inconsistency and acceptable variation

Different answers are not necessarily contradictory answers.

For open reasoning tasks, assurance must distinguish:

-   lexical variation;
-   reasoning-path variation;
-   capability-use variation;
-   outcome variation;
-   material contradiction;
-   boundary violation.

A probabilistic system should not be penalised simply for failing to
reproduce an identical trajectory.

### 7.5 Confidence versus correctness

Humans can be confidently wrong.

AI systems can produce fluent, confident outputs that are incorrect.

Observable confidence is therefore not itself evidence of competence.
Evaluation must compare claims, evidence, outcomes and repeat behaviour
rather than relying on presentation quality.

### 7.6 Behavioural drift

Perhaps the most operationally important dimension is change over time.

A system may continue functioning while its behaviour gradually moves
away from an established baseline.

Nothing crashes.

No exception is thrown.

Each individual execution may remain superficially plausible.

Yet assumptions can accumulate, context can shift, capability
utilisation can change, and behavioural patterns can move.

By the time the change becomes obvious, the first unambiguous evidence
may be an incident.

Reasoning Assurance should therefore ask not only:

> Did something fail?

but:

> **Is something changing that makes failure more likely?**

------------------------------------------------------------------------

## 8. From Experiment to Assurance: Lumen

Lumen operationalises these ideas as a chain from observation to
justified confidence.

The architecture should not be interpreted as a literal implementation
of human psychology. Rather, its components provide engineering
functions analogous to stages of empirical investigation.

### Moderari --- orchestration and preservation of conditions

Moderari governs execution and has access to important contextual
conditions surrounding model behaviour.

It preserves and exposes the information necessary for later assessment:
instructions, context, orchestration decisions, model/provider
information, tool availability and other execution conditions.

Moderari therefore contributes to understanding cognition in an
operational sense: not by declaring what an internal neural process
"meant", but by preserving the conditions and execution evidence from
which later behavioural interpretation becomes possible.

### Vestigare (Trace) --- observation

Vestigare records what happened.

It captures evidence of the execution rather than attempting to decide
whether the behaviour was good, bad, expected or significant.

This distinction is fundamental:

> **Observation is not assessment.**

### Repetere (Replay) --- reproduction

Repetere asks what happens when relevant conditions are repeated.

For deterministic software, divergence may itself indicate failure.

For probabilistic systems, reproduction instead allows us to begin
characterising behavioural range.

Repeated execution transforms an anecdote into evidence.

### Aestimare (Assess) --- interpretation

Aestimare asks whether observed differences matter.

It can classify divergence, assess task success, examine capability
utilisation, compare costs and evaluate whether behaviour remained
within acceptable boundaries.

This is where behavioural methodology becomes particularly relevant.

Aestimare should not merely ask whether two outputs are identical. It
should ask what kind of difference occurred and whether that difference
is material to the objective.

### Fiducia (Assurance) --- accumulation

Fiducia turns individual assessments into longitudinal evidence.

One successful execution establishes very little.

Repeated success under defined conditions begins to establish a
behavioural pattern.

Repeated evidence across varied conditions begins to establish
confidence.

Failures, boundary violations and behavioural changes revise that
confidence.

The result is not certainty.

It is **calibrated trust supported by accumulated evidence**.

The Lumen chain can therefore be expressed as:

**Observe → preserve → reproduce → compare → assess → accumulate →
assure**

------------------------------------------------------------------------

## 9. Behavioural Baselines, Not Expected Answers

Traditional testing commonly begins with an expected answer.

For many probabilistic reasoning tasks, that is too restrictive.

Consider a navigation system offering several routes to the same
destination. One route may be faster, another cheaper, another simpler.
They differ, but all satisfy the fundamental objective.

A route to the wrong destination is qualitatively different.

The same principle applies to AI reasoning.

Several executions may:

-   choose different reasoning paths;
-   use different tools;
-   emphasise different evidence;
-   produce differently structured outputs;
-   incur different computational costs;

while all remaining acceptable.

Reasoning Assurance therefore needs **behavioural baselines**, not
merely golden answers.

A behavioural baseline describes the observed and acceptable range of
behaviour under known conditions.

This allows us to ask:

-   Is the current execution within the established range?
-   Is a new type of behaviour appearing?
-   Is the frequency of a known behaviour changing?
-   Has capability utilisation shifted?
-   Are costs changing?
-   Are failures clustering differently?
-   Has a previously rare boundary condition become common?

This creates the possibility of detecting drift before it becomes
failure.

It also changes the meaning of regression testing.

For a probabilistic system, regression may not mean:

> The answer changed.

It may mean:

> **The behavioural distribution changed in a way that matters.**

------------------------------------------------------------------------

## 10. From Simon to Sufficient Trust

Herbert Simon provides another important bridge between human cognition
and Reasoning Assurance.

Simon challenged models of perfectly rational decision-makers operating
with unlimited information and computational capacity. His work on
bounded rationality recognised that real decision-making occurs under
constraints.

Humans have limited:

-   information;
-   attention;
-   time;
-   cognitive capacity.

AI systems operate under constraints too:

-   context windows;
-   computation;
-   latency;
-   energy;
-   financial cost;
-   tool availability;
-   information quality.

Assurance itself is also bounded.

We cannot test every possible prompt, context, tool combination, model
state, task and environmental condition.

Absolute proof of reliable behaviour across an effectively unbounded
possibility space is therefore not a realistic operational objective.

Simon's concept of **satisficing** offers a more useful framing.

The assurance question becomes:

> **Do we have enough evidence, for this task, under these conditions
> and given these consequences, to justify reliance on the system?**

The required evidence should depend on consequence.

A low-impact summarisation task may require relatively little historical
assurance.

A system permitted to take consequential financial, medical, legal or
infrastructure actions should require substantially stronger evidence.

This produces a practical relationship:

**greater consequence → greater required assurance**

and:

**greater delegated capability → greater need for justified trust**

Reasoning Assurance is therefore itself a bounded decision problem.

The objective is not perfect knowledge.

It is sufficient evidence for responsible reliance.

------------------------------------------------------------------------

## 11. Reasoning Assurance as an Empirical Discipline

The deeper implication is that assurance for probabilistic AI may
increasingly need to resemble experimental science.

Not because AI is human.

Not because an LLM has a psychology.

But because deterministic inspection is no longer sufficient to answer
the operational questions we need to ask.

The emerging methodology looks familiar:

1.  **Observe behaviour.**
2.  **Form a hypothesis.**
3.  **Preserve the relevant conditions.**
4.  **Repeat the experiment.**
5.  **Introduce controlled variation.**
6.  **Compare outcomes.**
7.  **Classify differences and failures.**
8.  **Attempt to falsify the hypothesis.**
9.  **Accumulate evidence longitudinally.**
10. **Revise confidence when new evidence appears.**

This is also why a successful execution is not enough.

One observation tells us what happened once.

Repeated observations begin to describe a distribution.

Controlled variation tells us something about robustness.

Failures tell us where boundaries may lie.

Longitudinal evidence tells us whether those boundaries remain stable.

Reasoning Assurance is therefore not the pursuit of certainty about an
inherently probabilistic system.

It is the disciplined accumulation and interpretation of evidence.

------------------------------------------------------------------------

## 12. Implications for Aestimare

This perspective has direct consequences for the future design of
Aestimare and its assessment corpus.

A useful corpus should contain more than tasks with definitive answers.

Definitive-answer tasks remain valuable because they establish a
controlled foundation. But behavioural assurance requires experiments
designed to expose and characterise variation.

Future Aestimare classes should investigate areas such as:

### Controlled framing variation

Present semantically equivalent tasks using different wording, ordering
or framing.

Ask whether behaviour changes and whether any change is material.

### Context interference

Introduce irrelevant, competing or misleading contextual information.

Measure whether established facts and objectives remain stable.

### Repetition and behavioural distribution

Repeat the same task under controlled conditions.

Measure the range of reasoning paths, capability usage and outcomes
rather than merely counting exact matches.

### Capability utilisation

Make tools or external capabilities available and examine whether the
model uses them appropriately and consistently.

A model that *can* use a capability does not necessarily *reliably
choose* to use it when appropriate.

### Perturbation testing

Change one relevant condition at a time:

-   system instruction;
-   model;
-   provider;
-   tool availability;
-   context;
-   orchestration policy.

Assess which behavioural properties remain stable and which change.

### Failure taxonomy

Do not merely record pass/fail.

Classify failure types.

A factual error, unsupported assumption, tool-selection error,
instruction violation, context loss and materially incomplete answer may
all represent different behavioural phenomena requiring different
responses.

### Longitudinal drift

Compare current behaviour with historical behavioural baselines.

The objective is to detect meaningful movement before an obvious
incident provides the first indication that behaviour changed.

These experimental classes align closely with Mitchell's call for
controls, stimulus variation, robustness testing and failure analysis,
while extending them toward an operational assurance objective.

Aestimare can therefore become more than an evaluator of individual
executions.

It can become an experimental instrument for characterising AI
behaviour.

------------------------------------------------------------------------

## 13. Conclusion

We may have spent too much time criticising artificial intelligence for
failing to behave like an idealised version of ourselves.

Human cognition is probabilistic, contextual, bounded, biased and
variable.

That does not excuse undesirable AI behaviour.

It tells us something about how we might study it.

Behavioural psychology and cognitive science have accumulated
generations of methodology for investigating systems whose internal
cognitive processes cannot be completely observed. Contemporary research
increasingly applies those methods to artificial intelligence, from
machine psychology to cognitive-science-inspired model evaluation.

Reasoning Assurance can build upon that foundation.

The goal is not to prove that artificial cognition is human cognition.

The goal is not to anthropomorphise the model.

The goal is not to accept error because humans also make errors.

The goal is to recognise that **probabilistic behaviour requires
empirical characterisation**.

Lumen's contribution is to carry that principle into operational
engineering:

**observe behaviour; preserve its conditions; reproduce it; assess
meaningful variation; accumulate evidence; revise confidence.**

The central distinction remains:

> **Borrow the methodology, not the ontology.**

And the central engineering question becomes:

> **What does the accumulated evidence justify us relying upon?**

Perhaps the mistake is not anthropomorphising AI.

Perhaps the deeper mistake is expecting probabilistic intelligence ---
human or artificial --- to behave like deterministic software.

------------------------------------------------------------------------

## References

1.  Tremblay, S., Marois, A., Zare, M., & Lafond, D. (2026). *Shared
    Minds: The Cognitive Parallels Between Humans and Artificial
    Intelligence*. Human Behavior and Emerging Technologies,
    2026, 9946143. https://doi.org/10.1155/hbe2/9946143

2.  Mitchell, M. (2026). *Six principles for evaluating cognitive
    capabilities in AI models*. AI Magazine, 47, e70061.
    https://doi.org/10.1002/aaai.70061

3.  Hagendorff, T., Dasgupta, I., Binz, M., Chan, S. C. Y., Lampinen,
    A., Wang, J. X., Akata, Z., & Schulz, E. (2024 revision). *Machine
    Psychology*. arXiv:2303.13988.
    https://doi.org/10.48550/arXiv.2303.13988

4.  Newell, A., & Simon, H. A. (1972). *Human Problem Solving*.
    Prentice-Hall.

5.  Simon, H. A. (1955). *A Behavioral Model of Rational Choice*. The
    Quarterly Journal of Economics, 69(1).

6.  Simon, H. A. (1980). *Cognitive Science: The Newest Science of the
    Artificial*. Cognitive Science, 4(1), 33--46.

7.  Simon, H. A. (1981). *Information-Processing Models of Cognition*.
    Journal of the American Society for Information Science, 32(5),
    364--377.

8.  Taylor, J. E. T., & Taylor, G. W. (2021). *Artificial Cognition: How
    Experimental Psychology Can Help Generate Explainable Artificial
    Intelligence*. Psychonomic Bulletin & Review, 28(2), 454--475.
    https://doi.org/10.3758/s13423-020-01825-5

------------------------------------------------------------------------

## Research Position

This paper presents a research direction for **Lumen Aestimare** and the
wider Lumen Reasoning Assurance architecture. It does not claim
equivalence between biological and artificial cognition. Its proposal is
methodological: established behavioural-science techniques may provide
useful experimental foundations for characterising probabilistic AI
behaviour and translating repeated observation into justified
operational confidence.
