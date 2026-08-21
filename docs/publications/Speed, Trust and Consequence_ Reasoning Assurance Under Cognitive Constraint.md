# Speed, Trust and Consequence  
## Reasoning Assurance Under Cognitive Constraint

### Abstract

Artificial intelligence is dramatically increasing the speed at which information can be gathered, synthesised, interpreted and acted upon. Human cognitive capacity, however, has not undergone a corresponding acceleration.

This creates an emerging asymmetry: **execution is becoming faster than comprehension**.

The conventional response is to insist upon continued human verification — keeping a human “in the loop”. But if machine output can be produced faster than humans can meaningfully evaluate it, universal human verification simply transfers the bottleneck from execution to assessment. The benefits of automation are progressively lost.

A more sustainable response requires **calibrated trust**: allowing systems to operate with differing degrees of human oversight according to accumulated evidence about their behaviour and, critically, the consequences of misplaced trust.

This paper considers an additional complication. An information system — whether a journalist, an analyst or an AI model — can present individually accurate facts while nevertheless inducing a conclusion that is not adequately supported by those facts. Factual accuracy is therefore necessary, but it may not be sufficient for reasoning assurance.

A contemporary news report concerning allegations against the late Cambridge academic Jason Arday provides a useful case through which to examine this distinction.

---

## 1. When Correct Information Produces an Unsupported Conclusion

Consider three propositions:

**A.** A person has been accused of plagiarism.

**B.** Questions have been raised about claims the person made concerning extraordinary athletic achievements.

**C.** Questions have been raised about claims concerning charitable fundraising.

Each proposition may have its own evidence.

But evidence supporting B is not necessarily evidence supporting A. Nor does evidence supporting C establish B.

Nevertheless, when A, B and C are presented together within a coherent narrative, something psychologically understandable can occur:

**A + B + C → this person appears to have a pattern of dishonesty.**

Once that contextual interpretation forms, the propositions can begin reinforcing one another.

Questions about athletic achievements make the plagiarism allegation feel more credible. The existence of a plagiarism allegation makes questionable fundraising claims seem more plausible. Those claims then contribute further to the impression that the plagiarism allegation is probably true.

The result can resemble corroboration even though the underlying propositions remain evidentially independent.

This distinction matters because **contextual coherence is not the same as evidential dependence**.

---

## 2. The Jason Arday Case

In August 2026, considerable media attention surrounded Professor Jason Arday following allegations concerning plagiarism, his academic history and aspects of his personal biography.

The University of Cambridge publicly stated on 2 August that allegations concerning Arday's thesis and some journal publications had been investigated by Liverpool John Moores University and relevant journals. On 5 August, Cambridge separately announced an investigation following new information concerning his academic qualifications and honorary appointments, while noting that other academic-misconduct complaints remained ongoing.

Arday subsequently resigned. Reporting at the time described a widening controversy in which allegations concerning plagiarism became accompanied by scrutiny of other aspects of his biography, including charitable and athletic claims.

Following Arday's death, the controversy itself became the subject of further scrutiny, including criticism of the intensity and nature of the public and media response.

A BBC News report concerning the suspension of an academic who had accused Arday of plagiarism provides an interesting example of the reasoning problem considered here.

The purpose of using this report is **not to determine whether its journalism was improper, nor to determine whether any allegation concerning Arday was true or false**.

The interesting question is narrower:

**What understanding might a reader reasonably construct from the way individually relevant facts are presented together?**

A reader encountering plagiarism allegations alongside questions about marathon running, fundraising or other biographical claims may reasonably perceive a common underlying explanation:

> If some extraordinary claims were unreliable, perhaps the plagiarism allegations were more likely to be true as well.

That is an intuitively understandable inference.

But it is still an inference.

Questions concerning one set of claims can legitimately affect a person's assessment of someone's general credibility. They do not, by themselves, establish whether particular passages of academic work constitute plagiarism. That proposition requires evidence relating to the academic work itself.

Indeed, Cambridge's own subsequent statement illustrates the importance of procedural separation. On 12 August, Vice-Chancellor Deborah Prentice said investigations needed to be thorough and transparent and specifically cautioned against the desire for quick answers and rapid responses at the expense of allowing reviews to complete their work effectively.

The case therefore illustrates a broader distinction:

> **Facts can be individually supported while the relationship a reader infers between those facts remains insufficiently supported.**

---

## 3. Factual Integrity and Reasoning Integrity

Most discussions of AI reliability concentrate on factual integrity.

Did the model invent something?

Did it cite a source correctly?

Was a numerical calculation accurate?

Did it correctly retrieve the requested information?

These are essential questions.

But imagine an AI system presenting propositions A, B and C above. Suppose every statement is factually correct and properly sourced.

A conventional factual assessment might therefore give the response a perfect score.

Yet the presentation might still strongly encourage:

**Therefore D.**

And D might not follow from A, B and C.

This suggests a distinction between two forms of assurance.

### Factual assurance

**Are the individual claims adequately supported?**

### Reasoning assurance

**Does the evidence presented adequately support the understanding or conclusion that the presentation encourages the recipient to form?**

The second is considerably harder.

It requires attention not merely to statements but to relationships between statements: relevance, independence, implication, omission, confidence and consequence.

---

## 4. Speed Changes the Problem

This issue is not unique to artificial intelligence.

Modern journalism operates under considerable pressure to research, synthesise and publish rapidly. Online news cycles continuously demand new information and updates.

Speed has enormous benefits. Information that once took days to distribute can now reach millions of people almost immediately.

But human cognitive capacity has not accelerated alongside information distribution.

AI increases this disparity dramatically.

An AI system can potentially:

**research → analyse → synthesise → recommend → execute**

in the time a human requires merely to read the resulting output.

The problem can therefore be expressed simply:

> **Execution speed has increased enormously. Comprehension speed has not.**

This creates an important limitation on the familiar idea of human oversight.

If every machine action requires meaningful human verification, then as machine throughput increases, the human becomes the limiting component.

Eventually one of two things happens.

Either automation slows to the speed at which humans can verify it, or verification becomes increasingly superficial.

Neither represents genuine assurance.

---

## 5. Bounded Cognition in a Machine-Speed Environment

Humans have always made decisions under cognitive constraints.

We cannot possess all relevant information, consider every possible alternative or calculate every consequence before acting. We therefore use heuristics, experience, institutional processes and trust to make decisions within practical limits.

AI does not remove those limits.

It may actually expose them more clearly.

A system capable of producing one carefully researched analysis per day gives a person considerable opportunity for scrutiny.

A system capable of producing hundreds of analyses, recommendations or actions in the same period does not.

The amount of information available to the decision-maker has increased.

The amount of attention available to evaluate it has not.

The result is a growing **assurance gap** between what machines can produce and what humans can independently verify.

Attempting to solve that gap by demanding more human attention cannot scale indefinitely.

The solution must therefore involve deciding **where human attention is actually necessary**.

---

## 6. Trust as a Mechanism for Reducing Cognitive Load

Trust is sometimes treated as the opposite of verification.

It is more useful to think of trust as the result of evidence that makes continual verification unnecessary.

Trust in an AI system need not mean:

> *I believe this system is correct.*

A more operational definition is:

> **I have sufficient evidence about this system's behaviour in this class of circumstances to accept the consequences of allowing it to operate without my immediate verification.**

This makes trust contextual rather than absolute.

We may trust the same system differently for different tasks.

An AI renaming variables in an internal software project requires relatively little assurance. Errors are inexpensive, visible and reversible.

An AI making a recommendation affecting someone's employment, liberty, finances or reputation requires considerably more.

The difference is not simply model capability.

It is **consequence**.

---

## 7. Consequence Determines the Required Assurance

A useful conceptual relationship is:

> **Required assurance ∝ uncertainty × consequence**

This is not proposed as a literal mathematical formula. It expresses a design principle.

As uncertainty rises, assurance requirements should rise.

As the consequences of error rise, assurance requirements should also rise.

A low-confidence answer concerning a trivial and reversible decision may require little intervention.

A high-confidence answer concerning an irreversible, high-consequence decision may still deserve considerable scrutiny.

This changes the purpose of human oversight.

Rather than:

> **A human must verify every AI action.**

we move toward:

> **Human attention should be allocated according to uncertainty, accumulated evidence and consequence.**

That is calibrated delegation.

---

## 8. Cause, Effect and the Propagation of Error

Consequence also requires considering what happens *after* an AI produces an output.

The relevant chain may look like:

**AI behaviour → output → interpretation → decision → action → downstream effects**

An error at the beginning of that chain may be trivial if nothing consequential depends upon it.

The same error may be extremely important if its output automatically determines subsequent actions.

This is why reasoning assurance cannot be concerned exclusively with whether an output is technically correct.

It must also ask:

- What interpretation is likely to follow?
- What decision might be based upon it?
- What systems or people might act upon that decision?
- How reversible are those actions?
- What is the consequence if the original reasoning was wrong?

The greater the propagation potential, the stronger the justification required for autonomous operation.

---

## 9. From Human-in-the-Loop to Human-When-Required

“Human in the loop” is frequently presented as the solution to AI risk.

For high-consequence decisions it may be essential.

But as a universal architecture it has an inherent scalability problem.

If humans must verify everything machines produce, machine-speed execution ultimately becomes human-speed execution.

The more sustainable objective is therefore not necessarily **human in the loop**.

It is:

> **Human when required.**

That requires evidence.

A possible assurance cycle becomes:

**Observe → Assess → Accumulate Evidence → Calibrate Trust → Delegate → Observe Consequences → Recalibrate**

Over time, behaviour that is consistently understood, predictable and low consequence can require progressively less human intervention.

Unexpected behaviour, changing circumstances or increasing consequences can trigger greater scrutiny.

Human attention therefore becomes a dynamically allocated resource rather than a mandatory checkpoint.

---

## 10. Returning to the News Example

This brings us back to the Arday reporting.

Suppose every factual statement within an article were independently verified.

That would establish factual integrity.

But consider the additional question:

> **What conclusion is a reader likely to form from the selection and proximity of those facts?**

If the reader concludes that an unresolved plagiarism allegation is probably true because unrelated claims appear questionable, the reasoning relationship deserves examination independently of the accuracy of each individual statement.

This does not establish that the journalist intended such an inference.

Nor does it establish that every reader will make it.

It simply demonstrates why assurance concerned with human understanding must look beyond individual facts.

The same principle applies to AI.

A model may produce no hallucinations whatsoever and still construct an argument whose organisation produces more confidence than its evidence warrants.

That is not principally a factual failure.

It is a **reasoning-assurance failure**.

---

## 11. A Small Experiment

There is a simple way to explore whether this phenomenon actually occurred for readers of the BBC report.

Read the original BBC article before considering the questions below.

First, recall your immediate impression of the plagiarism allegations.

How confident were you that they were substantially true?

Now separate the principal propositions.

The existence of plagiarism allegations is one proposition.

Questions concerning athletic achievements are another.

Questions concerning charitable fundraising are another.

Other disputed biographical claims constitute still others.

Ask which pieces of evidence actually bear upon each proposition and which primarily influence your broader perception of the person concerned.

Then return to your original judgement.

**Has your level of confidence changed?**

If it has, ask a second question:

> **Did any of the facts change, or did your understanding of the relationship between those facts change?**

There is no expected answer.

A reader may become less confident, more confident or remain exactly where they began.

All three outcomes are informative.

The purpose is not to persuade the reader toward a particular conclusion concerning Jason Arday.

It is to make the structure of their own inference visible.

---

## 12. Why That Matters for AI

If a reader's confidence changes without receiving additional factual evidence, something important has occurred.

The information has remained substantially constant.

What changed was the reader's understanding of the **relationships within the information**.

That suggests an important role for reasoning assurance.

Assurance need not merely provide more information.

Sometimes it should expose the structure connecting existing information:

**claim → evidence → inference → confidence → consequence**

This can improve a person's ability to determine whether the confidence generated by an answer is warranted by the evidence supporting it.

As AI increasingly performs synthesis rather than simple retrieval, this becomes more important.

The danger is not merely that AI will provide false information.

It is also that AI can provide large quantities of correct information, organised into highly coherent explanations, at a speed and scale that makes independent human reconstruction of the reasoning increasingly impractical.

Fluency and coherence can make conclusions feel inevitable.

Assurance must help distinguish **coherence from justification**.

---

## 13. Trust Is Not the Absence of Verification

There is an apparent paradox at the centre of AI assurance.

We want AI because it can perform cognitive and operational work faster than we can.

Yet the more work we delegate, the less practical it becomes for us to verify every result ourselves.

The answer cannot simply be more verification.

It must be better evidence about **when verification is necessary**.

Trust therefore becomes an engineering concern rather than merely a psychological one.

It is accumulated from observed behaviour.

It is bounded by context.

It changes when behaviour changes.

And it must be calibrated against consequence.

The objective of reasoning assurance is therefore not to enable humans to verify everything machines do.

It is:

> **to provide sufficient evidence to determine what humans no longer need to verify, given the consequences of being wrong.**

That distinction may become increasingly important as machine execution continues to accelerate beyond human comprehension.

---

## Conclusion

The acceleration of information and AI execution creates an unusual problem.

We can increasingly produce, analyse and act upon information faster than we can fully comprehend it.

Attempting to solve that problem through universal human verification merely relocates the bottleneck.

Sustainable delegation therefore requires calibrated trust.

But trust cannot depend solely upon factual accuracy. Individually correct facts can be arranged into a context that encourages conclusions more strongly than the underlying evidence warrants.

Reasoning assurance must therefore consider not only:

**Is this information correct?**

but also:

**What conclusion does this information encourage?**

**Does the evidence justify that conclusion?**

**How confident should we be?**

and ultimately:

**What happens if we are wrong?**

Speed makes trust necessary.

Consequence determines how much trust we can afford.

Assurance provides the evidence with which to decide.

---

### A Question for the Reader

If you read the original BBC report before reading this paper, think back to your initial reaction.

**Has your attitude toward any of the allegations changed?**

If so, it would be interesting to know *what* changed.

Did you encounter a new fact?

Or did you simply start seeing the relationship between the existing facts differently?

There is no preferred answer. The distinction itself is the point.