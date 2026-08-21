# Lumen: A Reasoning Assurance Service

*The ecosystem, services, architecture and thinking behind
evidence-based trust in AI reasoning*

------------------------------------------------------------------------

## From a Single Tool to a Family

For the first two weeks of its existence, "Lumen" meant one thing: the
orchestration engine that let a small model read files far larger than
its own context window. But as the project's research question evolved
--- from *"can the model remember?"* to *"can we observe how it
thinks?"* to *"can we reproduce and trust its behaviour?"* --- a single
component could no longer carry all of that responsibility without
becoming a monolith.

On **10 August 2026**, the diary records the decision that formalised
what had already been happening in practice:

> **Lumen is the complete family/stack.**

The original orchestrator was renamed **Lumen Moderari**, taking its
place as one specialist among several. Every new capability since has
been given its own name, its own port, its own codebase, and ---
critically --- exactly one job. A recurring architectural habit runs
through the whole diary: whenever a component quietly starts
accumulating a second responsibility, the team stops and asks whether
that responsibility actually belongs somewhere else. Replay wanting to
schedule its own repeated runs, Pontis being tempted to understand model
traffic, Servire being tempted to reimplement other components' UIs ---
each of these was caught and corrected the same way. That discipline is
why Lumen today is nine small, sharply-bounded services rather than one
large one.

Each service carries a Latin name chosen to describe its single
function:

  -----------------------------------------------------------------------
  Service                 Latin meaning           Role
  ----------------------- ----------------------- -----------------------
  **Moderari**            to moderate, direct     The orchestrator ---
                                                  reasons with the model

  **Vestigare**           to track, trace         Records everything,
                                                  interprets nothing

  **Repetere**            to repeat               Reproduces a recorded
                                                  opportunity to solve a
                                                  problem

  **Pontis**              bridge                  Bridges protocols,
                                                  sessions and
                                                  communication planes

  **Rogare**              to ask                  A conversational client
                                                  --- asks, never
                                                  orchestrates or tools

  **Fiducia**             trust, confidence       Turns repeated evidence
                                                  into calibrated trust

  **Servire**             to serve                The operational control
                                                  plane for the whole
                                                  stack

  **Praebere**            to provide, supply      Owns the model-provider
                                                  lifecycle

  **Aestimare**           to assess, value        The analytical engine
                                                  --- measures replay
                                                  outcomes objectively
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## The Guiding Principle

Long before any of these components had names, the project had already
arrived at the sentence that now governs how they fit together. The
individual component specifications each restate their own piece of it,
but laid end to end they form one continuous pipeline:

> Trace captures what happened. Replay reproduces what mattered. Assess
> determines what it means. Servire brings everything together.

Everything else --- Pontis, Rogare, Fiducia, Praebere, Moderari ---
exists to make that pipeline *possible* and *operable*: to give the
reasoning something to happen through, a session to travel over, a
control room to watch it from, a lifecycle to depend on, and a way to
decide how often it's worth checking again.

A second phrase, drawn from the ecosystem's own branding, describes why
any of this is worth building at all:

> **Light to see illuminates one.** Knowledge grows through observation.
> Understanding grows through experimentation. Insight grows through
> illumination.

------------------------------------------------------------------------

## Moderari --- The Orchestrator

Moderari is the direct descendant of the very first Lumen prototype: the
component that actually holds the conversation with the model. Its own
architecture document states its purpose in a single line:

> **Orchestrate the model-facing conversation while preserving Lumen
> session continuity and behavioural context.**

Moderari sits deliberately downstream of everything else in the request
chain --- by design, the *last* place a request passes through before
reaching the model:

    Rogare → Pontis → Vestigare → Repetere → Moderari → Model provider

alongside Praebere, which operates as a parallel lifecycle path rather
than a stop in the conversation itself:

                     Servire
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
         Moderari                Praebere
            │                       │
            │ model requests        │ provider lifecycle
            └───────────┬───────────┘
                         ▼
                   Ollama / Model

Its responsibilities are deliberately broad *within* the conversation
and deliberately narrow *outside* it: accepting OpenAI-compatible model
requests, maintaining the logical Lumen session identity
(`session-rogare-YYYYMMDD-HHMMSS` for a Rogare-originated conversation),
preserving conversation context and compaction behaviour across a
long-running exchange, coordinating tool-call continuations, and
generating the heartbeat and progress events that let a slow model
response be told apart from a genuinely stalled request. That session
identity has to survive multiple user turns, tool-result continuations,
Rogare being torn down and rebuilt as a UI, and ACP provider activity
happening underneath it through Pontis --- Moderari is explicitly the
*authoritative* holder of that context, while a client's own transcript
is only ever a redraw of it.

What Moderari is careful never to become is just as deliberate. It
doesn't own the user interface, the provider process, protocol bridging,
recording, replay, or assessment --- and, notably, it shouldn't need to
understand *how* any of those things work in order to do its own job:

> Moderari should understand the logical model conversation. It should
> not need to understand how Rogare is embedded in Servire; how Pi
> implements ACP; how Pontis maps ACP session identifiers; how Ollama is
> launched or terminated; how Vestigare persists traces; how Repetere
> evaluates replay matching; or how Assess/Fiducia judge or schedule
> behaviour.

Its relationship to Praebere is captured in a single dividing line that
recurs, almost word for word, throughout the whole ecosystem's
documentation:

> **Moderari orchestrates model use; Praebere manages model
> availability.**

------------------------------------------------------------------------

## Vestigare --- Observe

Vestigare ("Trace") is the ecosystem's memory. Its original
specification described it as *not* part of what was then called Lumen
and *not* part of Pi: it was designed as a fully independent application
sitting transparently between the two. That wording has since been
superseded by the architectural decision that **Lumen is the complete
family/stack**. Vestigare is therefore now one of the services that
comprise Lumen, while retaining the architectural independence that the
original specification was trying to protect: it observes the execution
path without becoming coupled to Moderari, Pi, or the behaviour it
records. Its founding purpose is stated with unusual bluntness:

> Capture every interaction between Pi and Lumen exactly as it occurred,
> without modifying either side of the conversation.

The reason it exists at all traces back to the observation that started
the whole research ecosystem: using the same model, the same prompt, the
same files, the same environment and the same Lumen version, answer
quality could still vary between runs --- and without an exact,
trustworthy recording of what actually happened, there was no way to
tell whether that variation came from different inputs, different
timing, different orchestration, or genuine model behaviour. Trace
exists purely to remove that ambiguity, and its rule is correspondingly
strict:

> It performs one responsibility only. **Observe.** It does not analyse,
> replay, modify, optimise, evaluate or interpret. It records. Nothing
> more.

Transparency is treated as close to an engineering invariant: Pi should
behave exactly as if it were talking directly to Lumen, and Lumen should
behave exactly as if Pi were talking directly to it --- the *only*
observable difference is that every message gets recorded. That means
preserving HTTP method, URL, query parameters, every header, every body,
status code and ordering, with nothing modified in either direction.

What gets preserved is intentionally simple and durable. Every recording
carries a user-supplied name with an automatic timestamp appended
(`operations.py baseline_20260801_084713`), and moves through one of two
small lifecycles:

    Idle → Recording → Completed
    Idle → Recording → Aborted

Only a *Completed* recording is ever eligible for replay. Underneath,
two MongoDB collections carry the data: `trace_recordings` holds
recording metadata (name, start/completion time, status), and
`trace_messages` holds every captured exchange --- recording ID,
sequence number, session ID, timestamp, direction, HTTP method, path,
headers, JSON body and request duration --- deliberately generic so the
schema stays valid as the rest of the ecosystem grows around it.

Vestigare's place in the pipeline is the first of four deliberate
stages:

    Lumen Trace → Lumen Replay → Lumen Assess → Lumen Servire

and its own definition of success is refreshingly concrete: Pi talks
through it unmodified, Lumen operates unchanged behind it, every request
and response is recorded and correctly stored, recordings can be
created, started, stopped and uniquely named, the interaction is visible
live --- and, once all of that holds, Vestigare becomes *"the trusted
source of experimental data for the remainder of the Lumen Research
Ecosystem."*

------------------------------------------------------------------------

## Repetere --- Repeat

Repetere ("Replay") began as an attempt to build something much simpler
than what it became: a system that would faithfully replay an entire
recorded conversation, tool calls included. That ambition broke the
first time a model chose a different tool mid-replay --- at that instant
Repetere no longer had the recorded results it needed to keep the
original conversation going, and reproducing Pi's entire runtime inside
Repetere just to solve that would have duplicated an enormous amount of
functionality for very little benefit.

Its founding specification states its scope with the same single-word
discipline as Vestigare's:

> Replay has one responsibility. **Reproduce.** Replay does not attempt
> to determine whether one answer is better than another. It simply
> recreates the same interaction under controlled conditions.

Concretely, that means selecting a completed Trace recording, creating
an isolated replay session, executing one or more replay runs against
it, resetting the model between runs, and recording replay metadata ---
all in service of one underlying research goal: understanding why an
identical AI system, given identical information, can still produce a
different answer. Typical experiments include simply repeating an
interaction, unloading and reloading the model between runs, or
comparing different model versions, inference parameters, or future
Lumen releases against a fixed baseline.

That original scope evolved twice as the architecture matured. First,
Repetere's purpose was redefined around a single research question
rather than literal reproduction:

> **How long does the model continue to follow the original behavioural
> path before choosing a different one?**

It replays not the *conversation*, but the **original opportunity to
solve the problem** --- the same system prompt, user prompt, model,
environment and tools --- and watches for the first meaningful
difference: a different tool, different arguments, an extra or missing
call, a different final answer. That moment is called the **Fork
Point**. Once reached, Repetere's job is finished: it logs the last
matching step and the first divergent one, then becomes completely
transparent, handing traffic back to the normal stack so Vestigare
resumes recording and the model's new, divergent path continues exactly
as if Repetere weren't there.

Second, Repetere was redefined a further time to compare only
*behaviour*, not *presentation*. Early versions still compared streamed
text, reasoning summaries, formatting, timing and Moderari's own
checkpoints --- none of which represent anything the model actually
*did*, and all of which could cause a false fork even when the model
behaved identically underneath. The principle that resulted is now
Repetere's permanent charter:

> **Replay compares behaviour, not presentation.**

A final, later boundary was drawn once Fiducia existed: Repetere
executes exactly **one** replay run per invocation. Deciding *when* and
*how many times* to run that experiment was deliberately pulled out and
given to Fiducia --- a clean split between performing an experiment and
orchestrating experiments, and one that the original specification's
language ("one or more replay runs") had not yet anticipated.

Repetere's place in the pipeline stays fixed regardless of these
refinements:

> Lumen Trace captures reality. Lumen Replay reproduces reality. Lumen
> Assess evaluates the results. Replay forms the bridge between
> observation and analysis.

In everyday operation it is an inert pass-through in the request chain,
only activating on an explicit operator command (`\obt replay start`),
which means it can stay permanently deployed without ever touching an
ordinary conversation.

------------------------------------------------------------------------

## Pontis --- Bridge

Pontis is the component that turned out to be architecturally necessary
almost by accident. It began from a wrong assumption --- that a protocol
called ACP (Agent Client Protocol) would carry entire conversations
between Servire and Pi --- and that assumption broke during real
testing. What actually happens is more elegant: ACP only *starts* a Pi
session; the moment it's bootstrapped, Pi falls back to its normal
OpenAI-compatible HTTP interface to talk to whatever it believes is its
model provider. Because Pontis is configured as that provider, it ends
up simultaneously acting as an ACP client of Pi *and* the HTTP endpoint
Pi is talking to --- two independent communication planes meeting in one
component.

Pontis's own architecture document is explicit about why it exists at
all: as Lumen's responsibilities separated out --- orchestration to
Moderari, observation to Vestigare, reproduction to Repetere, evaluation
to Assess, lifecycle to Servire --- one responsibility remained
unclaimed: understanding client protocols and session management. Rather
than let that understanding leak into Vestigare or Repetere, Pontis
exists to isolate it entirely.

> During normal operation Pontis behaves as a transparent bridge.
> Replay, Trace and Lumen remain unaware that Pontis exists.

Four design goals hold that in place: keep the behavioural engineering
stack client-agnostic, forward messages transparently during normal
operation, enable advanced session operations without ever modifying
clients, and support multiple client protocols through one common
abstraction. Its position places it first in line for all client
traffic:

    Client → Pontis → Trace → Replay → Lumen → Model / Tools

Its responsibility list is broad on the *client* side of the boundary
--- client protocol integration, session management, client identity,
message forwarding, session routing, capability negotiation, client
reconnection, session migration, replay reinjection, tool-provider
bridging --- and its non-responsibility list is, deliberately,
everything that touches behaviour itself: Pontis never records, replays,
compares, analyses, scores, manages checkpoints, owns continuity,
orchestrates models, or assesses behaviour.

Pontis's most consequential capability is **lazy provider session
attachment**. When conversational traffic arrives for a session with no
live tool-provider connection behind it --- exactly the situation the
moment Repetere forks --- Pontis is the component responsible for
quietly establishing that connection (today, a Pi ACP session) so the
model's new, divergent path can continue talking to real tools without
Repetere ever having to know how a provider session gets created. Replay
delegates reinjection to Pontis and remains completely unaware of client
protocols; Pontis, in turn, understands exactly how to resume the
conversation for whichever client is attached.

Pontis is designed around **client adapters** rather than a fixed
protocol --- Pi today, with ACP, Zed ACP, VS Code, Cursor, Claude
Desktop, the OpenAI Responses API and Anthropic's own APIs named as
future candidates --- with the explicit design promise that adding a new
adapter should require no changes to Trace, Replay or Lumen. The same
reasoning is expected to eventually extend to tool providers themselves,
kept just as isolated from the behavioural engineering stack.

Four principles are named directly in its specification: **Single
Responsibility** (Pontis bridges sessions, nothing more),
**Transparency** (a normal conversation should look exactly as if Pontis
weren't there), **Client Isolation** (client-specific behaviour must
never leak into Trace, Replay or Lumen), and **Extensibility** (new
clients arrive as adapters, not modifications). One carve-out exists
purely for its own development: during standalone testing, Pontis may
terminate requests and return deterministic fixture responses to verify
its protocol handling in isolation --- but the specification is emphatic
that these are test fixtures only, never part of its real operational
behaviour. The invariant that ties all of this together:

> **During development, Pontis may terminate requests for testing.
> During normal operation, Pontis never becomes the conversational
> endpoint. It transparently bridges client communication into and out
> of the Lumen stack.**

------------------------------------------------------------------------

## Rogare --- Ask

Rogare is Lumen's own conversational client, and its architecture
document opens with the same single-sentence discipline as everything
else in the family:

> **Provide the human conversational surface without owning model
> orchestration, tool execution, or conversational context.**

Its position in the stack makes explicit something easy to miss
elsewhere: Rogare talks *only* to Pontis. It never reaches Pi, Moderari
or the model provider directly ---

    Servire (embedded / pop-out)
            │
            ▼
         Rogare
            │
       OpenAI-compatible HTTP
            │
            ▼
         Pontis
        ┌────┴────┐
       ACP       HTTP
        │          │
        ▼          ▼
        Pi    Vestigare → Repetere → Moderari → Provider
        ▲                              │
        └──────── tool path ───────────┘

Rogare owns the parts of the experience a person actually sees and
touches: starting and naming sessions, accepting prompts, presenting
responses, showing session and provider state, displaying heartbeat and
progress information, and supporting both embedded and pop-out
presentation inside Servire's workspace. It deliberately owns none of
what happens underneath: model selection or lifecycle, tool execution,
ACP protocol management, model orchestration, or --- most importantly
--- the authoritative conversational context itself.

> **Rogare displays the conversation; Lumen owns the conversational
> context.**

A Rogare session carries a logical identifier
(`session-rogare-<timestamp>`) that Pontis propagates into the stack,
and Pontis alone owns the correlation between that identity and any
separate ACP/Pi session running underneath it --- Rogare is not expected
to know or care that the ACP session even exists. This distinction is
what makes Rogare's session-reattachment behaviour possible: on load, a
Rogare UI checks for an active session and reattaches to it ---
restoring provider state and redrawing the existing conversation ---
rather than silently starting a new one. A genuinely new conversation
only ever begins through an explicit `New Session` action, which
detaches the current UI without requiring the underlying Lumen context
to be destroyed. This was a direct fix for a real regression: switching
Rogare between its pop-out window and Servire's embedded workspace had
been quietly orphaning a live session by unconditionally preparing a new
one on load.

Provider session establishment happens through Rogare but is not
performed by it: starting a session causes Pontis to create the
necessary ACP session with Pi, select the configured provider/model, and
run a provider initialisation request through the model path --- all
before Rogare marks the session ready, which is why a cold model start
can visibly take much longer than a warm one. Heartbeat and progress
information travels the same backchannel route already established by
Moderari and Pontis, with Rogare presenting request-elapsed time,
provider progress text, and estimated context usage by polling rather
than requiring a persistent stream.

When tools are needed, Rogare's own description of the flow makes its
non-involvement explicit:

    Rogare → Pontis → Moderari/model reasoning → (tool requirement) → Pontis ACP bridge → Pi → Tool

Rogare's architectural invariants close the loop: it is a client, not an
orchestrator; it must never become the authoritative owner of context
simply because it renders the transcript; UI recreation, embedding or
reload must never implicitly start a new logical conversation while a
session can be reattached; and it must not duplicate the
responsibilities of Pontis, Moderari, Praebere or Pi. That separation is
precisely what lets Rogare exist as *a* Lumen conversational experience
rather than *the* Lumen reasoning architecture.

------------------------------------------------------------------------

## Fiducia --- Trust, Earned

Fiducia began, and for a long time remained, only an idea --- a "future
concept" entry the team deliberately parked until the rest of the
ecosystem matured. Its name was chosen for exactly what it needed to
represent:

> The name *Fiducia* (Latin) conveys **trust, confidence and
> assurance**. It reflects confidence that has been earned through
> evidence rather than assumed.

Fiducia's founding purpose is unusual among the nine services in one
respect: it was conceived from the start as *not* participating in
normal user interactions at all. Instead it would operate alongside the
rest of Lumen as an independent assurance layer, periodically validating
that the combined Lumen--model system continues to behave within
empirically measured baselines --- not to make the model deterministic,
but to provide ongoing evidence that its behaviour remains consistent
with what's already been observed. Its originally imagined
responsibilities read as a checklist of everything the rest of the
ecosystem would eventually need to be composed together to deliver:
periodically execute approved Replay plans, compare current behaviour
against historical baselines, draw on Trace data as supporting evidence,
invoke Assess to evaluate behavioural differences, detect drift
following a model or Lumen change, and produce assurance reports and
continuity metrics over time --- a continuously evolving picture of
model behaviour rather than a set of isolated benchmark results.

One early design idea worth preserving is that Fiducia was imagined,
from its very first sketch, as a candidate for an **agentic workflow**
--- a bounded AI agent responsible for coordinating its own assurance
loop (scheduling tests, collecting evidence, producing reports) while
remaining outside the normal Lumen request path, with human review
deliberately retained for any significant conclusion or operational
decision. The idea was explicitly framed as a chance to gain practical
experience with agentic design while staying fully aligned with Lumen's
broader emphasis on observability and provenance.

By the time Fiducia moved from concept to build, its role had sharpened
into something more concrete and better bounded: it does not perform
experiments itself --- Repetere already does that, one run at a time.
Fiducia decides **when**, **how often**, and **how many times** those
experiments should run, and accumulates the evidence needed to know how
much verification a given behaviour still deserves:

> **Repetere executes one Replay Run. Fiducia decides when, how often
> and how many Replay Runs should occur.**

This split traces back to the same instinct that shaped every other
boundary in the ecosystem: an early plan to let Repetere itself support
"run this replay X times" was rejected precisely because it would have
made one component responsible for both performing an experiment and
orchestrating experiments. In practice, Fiducia runs on recurring
schedules; each occurrence creates a **Fiducia Execution**, and a
separate `runs_per_execution` setting controls how many individual
Repetere Runs that execution requests. Fiducia was also built to behave
sensibly around its own downtime --- it doesn't try to catch up by
firing every missed occurrence on restart --- and to recover incomplete
executions cleanly rather than leaving orphaned state behind.

Fiducia is best understood not as another stop in the conversational
execution pipeline, but as the service that closes the **Lumen Assurance
Loop**. It drives experimentation forward by scheduling Repetere, while
Aestimare feeds measured evidence back into Fiducia:

                        Fiducia
                       /       ▲
              schedules        │ assessment
                     /         │
                    ▼          │
                Repetere ──► Aestimare
                       evidence

The two directions have deliberately different responsibilities:

-   **Fiducia → Repetere:** decide when, how often, and how many replay
    experiments should run.
-   **Repetere → Aestimare → Fiducia:** turn the resulting behavioural
    evidence into measurement, and measurement into an increasingly
    calibrated basis for trust.

Fiducia can already operate on Repetere's raw outcomes and execution
history before Aestimare is fully developed. As Aestimare matures, the
evidence returned to Fiducia becomes richer than a simple
matched/divergent result:

    Repetere → behavioural evidence → Aestimare → assessment → Fiducia → calibrated trust

This lets Fiducia be built and proven incrementally while preserving the
long-term architecture. Its founding statement still governs the whole
idea, unchanged from the day it was first written down:

> **Trust isn't the absence of verification. Trust is what accumulated
> evidence allows you to stop verifying every time.**

Even the earliest branding notes reflect this: Fiducia's proposed logo
was a modern, minimalist set of balance scales --- chosen specifically
to represent *weighing evidence*, comparing expected behaviour against
observed behaviour, rather than anything resembling a legal or punitive
judgement.

------------------------------------------------------------------------

## Aestimare --- Assess

Where Repetere answers *"did behaviour change?"*, Aestimare exists to
answer the question Repetere was always careful never to touch:

> **Was the new behaviour better, worse, equivalent, or simply
> different?**

Aestimare is the analytical component of the ecosystem. It does not
execute experiments --- that's Repetere's job --- it evaluates their
*results*. Its founding design philosophy is deliberately austere:

> Assess has one responsibility. **Measure.**

That single word carries a specific methodological commitment:
assessment should always begin with deterministic, reproducible
measurement before any semantic interpretation is introduced. In
practice this means Aestimare's first generation of capability is
intentionally simple --- are two answers identical or different, what do
their hashes look like, how do basic response statistics compare ---
precisely so that every experiment has an objective, repeatable baseline
before anything more subjective is layered on top. This mirrors, almost
exactly, the lesson Moderari's checkpoint work learned the hard way in
its own early history: a component should preserve and report what it
can actually verify, not reach for a judgement it can't yet justify.

The reason Aestimare needs to exist at all is one of scale. A single
Repetere run is easy for a person to read and judge by eye; a Fiducia
schedule running that same experiment across hundreds or thousands of
occurrences is not. Aestimare exists to make that volume of behavioural
evidence tractable --- comparing replay outputs, identifying variation,
and producing comparison reports and experiment summaries an operator
can actually act on.

Its scope is explicitly bounded, in the same style as every other Lumen
component: Aestimare does not generate AI answers, and it does not
decide how or when experiments run. Later capability is expected to grow
toward genuinely semantic comparison --- checkpoint comparison,
reasoning-path evolution, evidence coverage, capability utilisation,
even AI-assisted assessment --- but however sophisticated it becomes,
Aestimare remains, by design, an independent analysis engine rather than
an experiment runner or a scheduler.

This closes the loop the whole ecosystem was built around:

> Trace records. Replay reproduces. Assess measures. Servire presents.

And it completes Fiducia's longer-term evidence chain: once Aestimare is
producing real measurements, Fiducia stops relying solely on raw replay
outcomes (matched/divergent) to decide how much verification a behaviour
still needs, and starts using Aestimare's actual judgements of *how
much* a divergence mattered --- the point at which "trust built from
evidence" stops being a slogan and becomes something genuinely measured.

------------------------------------------------------------------------

## Servire --- Serve

Servire is the one component in the family that deliberately stays
**outside** the model traffic path. While every other service sits
somewhere along
`Client → Pontis → Vestigare → Repetere → Moderari → Model Provider`,
Servire's job is to operate and observe that stack from the outside,
never to sit inside it. Its own founding philosophy is as austere as
Vestigare's or Aestimare's:

> Servire has one responsibility. **Serve the operator.** It
> orchestrates user interaction. It does not duplicate the
> responsibilities of Trace, Replay or Assess.

The reason it exists is almost a direct consequence of the rest of the
ecosystem's single-responsibility discipline: splitting Lumen into
focused, independently testable projects keeps each one simple, but a
researcher shouldn't have to juggle several unrelated user interfaces
just to run one experiment. Servire's early vision imagined it as *the*
place from which the research ecosystem is observed and managed ---
providing access to Trace, Replay and Assess in one dashboard, with
model lifecycle management, experiment history, system health, reporting
and administration named as natural future extensions.

That vision matured, in practice, into a specific architectural decision
that turned out to matter more than any single feature: Servire would
**host** other components' operator interfaces rather than
**reimplement** them. Each service still owns its own UI, its own logs,
and its own lifecycle behaviour; Servire simply federates them into a
single navigable workspace, built entirely from configuration, so that
new components can appear automatically without any change to Servire's
own code. This produced a genuine shift in how the team worked day to
day: instead of opening Lumen, Repetere, Vestigare and Servire as four
separate applications, an operator now simply opens **Servire**.

Servire's operational responsibilities kept expanding around that same
"operate, don't absorb" boundary: a unified, filterable, exportable
Operational Log merging every managed component's output into one
chronological stream; credential redaction after MongoDB connection
strings were briefly exposed in its dependency view; a graceful
full-stack shutdown sequence ensuring that stopping Servire also cleanly
stops everything it manages, down to releasing the loaded model; and a
stated ambition to manage not just service *lifecycle* but service
*topology* --- for instance, a future profile in which disabling
Vestigare automatically reroutes Pontis directly to Moderari without
Repetere being involved at all:

> **Servire should manage the topology of Lumen, not merely the
> lifecycle.**

Servire's own branding ties the whole ecosystem's purpose together under
a name the team calls *Illuminates.One*:

> **Light to see illuminates one.** Knowledge grows through observation.
> Understanding grows through experimentation. Insight grows through
> illumination.

Servire, in that framing, is not just a dashboard --- it exists
specifically to *illuminate the complete research journey*, from a raw
Trace recording through to a Fiducia-scheduled, Aestimare-measured
judgement about whether a model's behaviour can still be trusted.

------------------------------------------------------------------------

## Praebere --- Provide

Praebere is the newest and narrowest of the nine services, created
specifically so that Servire --- and Moderari --- would never need to
understand model-provider mechanics. Its architecture document states
its single responsibility plainly:

> **Manage the availability and lifecycle of the configured external
> model provider without owning conversational orchestration.**

Praebere sits alongside the conversational path rather than inside it
--- Servire controls its lifecycle, Praebere controls the provider's,
and Moderari simply sends model requests into whatever Praebere has made
ready:

                             Servire
                                │
                        lifecycle control
                                │
                                ▼
                        Lumen Praebere
                                │
                       provider lifecycle
                                │
                                ▼
                             Ollama
                                │
                                ▼
                        Configured Model

    Rogare → Pontis → Vestigare → Repetere → Moderari ──────┘
                             model request path

Its responsibilities are entirely about *availability*, never about
*reasoning*: starting the configured provider process, checking
availability, discovering or confirming the configured model, loading or
warming it, reporting provider and model state, and stopping only the
resources it itself manages. It never inspects prompts, responses,
conversation history, tool calls, session context, or replay/assessment
behaviour --- Moderari sends conversational traffic straight to the
provider endpoint, and Praebere never mediates or even looks at that
traffic. The dividing line is stated, again, as one clean sentence:

> **Moderari decides how the model is used. Praebere ensures the
> configured model provider is available to be used.**

A deliberate ownership boundary keeps this honest: if Praebere itself
started a provider process, it may stop it during shutdown; a provider
that happens to already be running externally is never assumed to be
Praebere's to terminate unless configuration explicitly grants that
ownership. For the current local deployment this means starting Ollama
(`ollama serve`), confirming it through its own HTTP API (`/api/tags`,
`/api/ps`), and preparing the configured model (`qwen2.5-coder:14b-32k`)
so it's ready before any conversational request needs it.

Cold starts are treated as a known, named condition rather than an
ambiguous delay: loading a model can allocate substantial memory before
settling to a lower steady state, and a Rogare session can legitimately
sit in *"Establishing provider session"* for a while even though every
Lumen service is otherwise healthy --- Praebere owns that portion of the
wait, while Pontis and Rogare own how it's presented to the person
watching. Praebere is also explicit about the difference between a
*running process* and a *ready provider*: process existence, provider
API reachability, model availability and model readiness are tracked as
separate states, specifically so Servire can present something more
honest than "it's on."

Shutdown follows the same ownership logic in reverse. Servire requests
lifecycle shutdown *from* Praebere rather than ever killing Ollama
directly, because only Praebere knows which provider is configured, how
it was started, whether it's actually Praebere's to stop, and how its
resources should be released. Provider shutdown is allowed to take
meaningfully longer than an ordinary Lumen service's --- Servire's
timeout for Praebere was extended from 180 to 300 seconds after real
testing found the default insufficient --- with an explicit note that if
shutdown keeps approaching that ceiling, the right response is to
investigate Praebere's own provider-shutdown behaviour, not to keep
raising the timeout.

Its architectural invariants close the same way every other component's
do:

> Praebere manages providers; it does not orchestrate conversations.
> Moderari must remain provider-agnostic at the lifecycle level. Servire
> coordinates lifecycle but delegates provider-specific start/stop
> behaviour to Praebere.

The payoff is explicit in its own documentation: the underlying model
supplier can change in the future without any provider-specific
lifecycle logic ever needing to move into Moderari, Rogare, Pontis, or
Servire.

------------------------------------------------------------------------

## How It All Works Together

The nine services are easiest to understand as three cooperating planes
rather than as nine boxes in one linear chain.

### The Execution Plane

This is the path a live conversation follows:

    Rogare / Client
          │
          ▼
        Pontis
          │
          ▼
      Vestigare
          │
          ▼
       Repetere
          │
          ▼
       Moderari
          │
          ▼
    Model Provider

**Praebere** sits alongside this path rather than inside it. It makes
the configured model provider available before Moderari needs it, and
manages that provider's lifecycle without seeing or mediating
conversational traffic.

During ordinary execution, Pontis bridges the client and session
boundaries; Vestigare records transparently; Repetere remains an inert
pass-through; and Moderari owns the logical model conversation and
reasoning continuity.

### The Assurance Plane

The evidence created by execution feeds a separate assurance process:

    Vestigare → Repetere → Aestimare → Fiducia
      observe      reproduce      measure       trust

But this is not merely a one-way pipeline. Fiducia schedules further
Repetere experiments, creating a feedback loop:

                        Fiducia
                       /       ▲
              schedules        │ assessment
                     /         │
                    ▼          │
                Repetere ──► Aestimare
                       evidence

This is the **Lumen Assurance Loop**.

Vestigare supplies faithful evidence of what happened. Repetere
recreates the original opportunity and identifies behavioural
divergence. Aestimare measures whether and how the resulting behaviour
differs. Fiducia accumulates those measurements over repeated
experiments and uses the growing evidence base to determine how much
continued verification a behaviour warrants.

Lumen therefore does not attempt to make probabilistic AI deterministic.
It treats behavioural variation as something to be observed and studied.
The assurance question is not *"how do we force the model to take
exactly the same path every time?"* but rather:

> **How do we observe variation, reproduce the conditions under which it
> occurs, measure its significance, and determine whether accumulated
> evidence gives us sufficient reason to trust the resulting
> behaviour?**

### The Operations Plane

**Servire** sits outside both conversational execution and behavioural
assessment:

                        Servire
                           │
              operates / observes / coordinates
                           │
                  Lumen ecosystem

Servire brings the stack up in dependency order, federates the
component-owned operator interfaces, presents operational state and
logs, coordinates shutdown, and is evolving toward managing deployment
topology as well as lifecycle. It operates the system without absorbing
the responsibilities of the services it manages.

Together, the three planes describe Lumen as a complete system:

-   **Execution** gives AI reasoning a controlled, continuous path
    through the system.
-   **Assurance** turns observed behaviour into reproducible
    experiments, measurement, and accumulated evidence.
-   **Operations** makes the distributed service family manageable as
    one coherent whole.

This separation is important because it prevents operational concerns,
conversational concerns, and assurance judgements from contaminating one
another while still allowing them to cooperate through explicit
interfaces.

------------------------------------------------------------------------

## The Thinking Behind the Shape

Two ideas recur often enough across both the diary and every component's
own specification to count as Lumen's real design philosophy,
independent of any single service:

**Single responsibility, aggressively enforced --- and stated
explicitly, every time.** It is striking how consistently each
component's own documentation opens by naming its one job in a single
sentence, and closes by naming everything it deliberately refuses to
become. Vestigare "observes --- nothing more." Repetere "reproduces" and
never judges. Pontis "bridges sessions --- nothing more." Servire
"serves the operator" and never duplicates Trace, Replay or Assess.
Praebere "manages providers; it does not orchestrate conversations."
This isn't incidental phrasing --- it's the same architectural
correction, applied component by component: Repetere shouldn't schedule
itself (that's Fiducia); Servire shouldn't reimplement other UIs (it
hosts them); Pontis shouldn't understand model or tool traffic (it just
bridges sessions); Moderari shouldn't need to understand how Vestigare
persists data or how Repetere matches behaviour. The team's own summary
of this pattern is worth repeating:

> Whenever a component starts accumulating an additional responsibility,
> it is worth asking whether that responsibility actually belongs
> elsewhere.

**Trust as an earned, evidence-based state, not an assumption.** This is
Fiducia's entire reason for existing, but it's really a statement about
the whole ecosystem: Lumen doesn't try to make a model deterministic,
and it doesn't take a single successful run as proof that a system is
safe to rely on. It builds instruments --- Vestigare to see, Repetere to
test, Aestimare to measure, and Fiducia to accumulate --- so that
confidence in an AI system's behaviour is something earned through
repeated, observable evidence, rather than something granted by default
because an answer looked right once.

Taken together, that is what turns Lumen from an orchestration engine
into a **Reasoning Assurance Service**.

Externally, Lumen provides one coherent capability: reasoning assurance.
Internally, that capability is implemented as an ecosystem of nine
sharply bounded services, each responsible for one part of execution,
evidence, measurement, trust, provider availability, or operation.

Lumen does not try to make AI more intelligent, nor does it try to
eliminate the variability inherent in probabilistic reasoning. It makes
AI behaviour **observable, reproducible and measurable, so that trust
can be based on evidence rather than assumption**.

That is the distinction at the centre of the system: observability can
tell us what happened; Lumen is being built to determine whether what
happened matters, how consistently acceptable behaviour can be
reproduced, and how much evidence is sufficient before confidence
becomes justified.
