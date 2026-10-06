# 1. Start with Minimal Moderari

Given what Lumen has become, I think the irreducible Moderari is considerably smaller than today's implementation.

Its fundamental position is:

```text
Pontis
   │
   │ model request
   ▼
┌─────────────────────────┐
│        Moderari         │
│                         │
│  Prepare model request  │
│           │             │
│           ▼             │
│       Provider          │
│           │             │
│  Interpret response     │
│           │             │
└───────────┼─────────────┘
            │
            ▼
          Pontis
```

And I would initially give that minimal Moderari only **three responsibilities**:

```text
1. Receive the model interaction
2. Establish the effective model-facing request
3. Translate the model response into Lumen/client-compatible form
```

That's it.

In more concrete terms:

```text
INPUT
  │
  │ model + messages + tools + execution/session identity
  ▼
MODERARI
  │
  ├── establish effective messages
  ├── apply model-specific adaptation
  └── call provider
  │
  ▼
MODEL
  │
  ▼
MODERARI
  │
  └── normalize provider/model response
  │
  ▼
OUTPUT
```

That gives us a clean definition:

> **Moderari controls the boundary between Lumen's model-independent interaction representation and the effective representation understood by a particular model/provider.**

That remains a compelling reason for Moderari to exist.

---

# 2. Then put today's functionality around that core

From the code we just examined, I would divide the existing Moderari responsibilities into roughly these groups:

| Capability | Minimal Moderari? | Re-evaluate? |
|---|---:|---:|
| OpenAI request handling | **Yes** | Implementation |
| Provider invocation | **Yes** | Especially with multiple providers |
| Model-specific request adaptation | **Yes** | Architecture |
| Model response translation | **Yes** | Architecture |
| System prompt policy | Probably | Scope |
| Model profiles | Probably | Ownership |
| Context measurement | Probably | Scope |
| Checkpoint generation | Maybe | **Major review** |
| Checkpoint application | Maybe | **Major review** |
| Session persistence | Questionable | **Major review** |
| Session restoration | Questionable | **Major review** |
| Read/file workflow supervision | Questionable | **Major review** |
| Tool replay suppression | Questionable | Review |
| Recovery nudges | Questionable | **Major review** |
| Operational state | Partly | Ownership |
| Pontis heartbeat/backchannel | Probably not core | Review |
| `\obt` command handling | Probably not core | Review |
| UI | Definitely not core | Review |
| Result persistence | Questionable | Review |
| Protocol evidence | Yes conceptually | Possibly Vestigare |
| Prompt provenance | Yes conceptually | Possibly Vestigare |
| Provider configuration discovery | No | Already moving elsewhere |
| Model residency | No | Already Praebere |

And that table immediately exposes why Moderari feels complicated.

It has accumulated responsibilities from **several generations of Lumen architecture**.

---

# 3. I think there are actually five Moderaris hiding inside Moderari

## A. Model Adapter

This is unquestionably Moderari.

```text
Lumen representation
        │
        ▼
 model/provider adaptation
        │
        ▼
Provider/model
        │
        ▼
 response normalization
        │
        ▼
Lumen representation
```

Qwen returning textual JSON tool requests that need converting into OpenAI `tool_calls` is a perfect example.

This capability remains valuable even as providers multiply.

In fact, it becomes **more** important.

---

## B. Model Context Constructor

This is also a strong candidate for Moderari:

```text
client history
system-prompt policy
checkpoint/continuity
tool definitions
model characteristics
        │
        ▼
   MODERARI
        │
        ▼
effective model context
```

This answers:

> **What exactly should this model see for this turn?**

That feels very much like Moderari.

And importantly, it is different from conversation ownership.

Pontis/session/client may own what *happened*.

Moderari owns what the **model needs to see now**.

That is a very clean boundary.

---

## C. Continuity Manager

This is where I think we need a serious rethink.

Today's Moderari:

```text
observe context
     │
checkpoint
     │
persist checkpoint
     │
replace historical model context
     │
restore sessions
     │
continue execution
```

Some of that clearly belongs near model-context construction.

But persistence, session resurrection and execution lifecycle may no longer belong there.

The distinction I would investigate is:

```text
Moderari:
    HOW do I construct a viable model context?

versus

Elsewhere in Lumen:
    WHAT is this session's authoritative continuity state?
```

Those are not necessarily the same responsibility.

---

## D. Agent/Workflow Supervisor

This is the part I am least convinced belongs in Moderari anymore.

Things such as:

```text
Did the model read all the files?

Did it stop too early?

Should I tell it to continue?

Has this tool already been executed?

Should I suppress that tool call?

Should I issue a recovery nudge?

Did it actually finish the requested job?
```

These are not really model adaptation.

They are **execution supervision**.

And Lumen today has Pontis, Fiducia, Aestimare and much richer trace/evidence infrastructure than existed when some of this Moderari behaviour was introduced.

So I would put a large red circle around:

> **Read workflow, recovery, completion detection and tool-execution supervision.**

Not because they're bad ideas—the opposite. They may be valuable Lumen capabilities.

But we should ask whether **Moderari is still the correct authority for them**.

---

## E. Observability / Control Plane

Today's Moderari also contains:

```text
operational state
results
sessions
checkpoint UI
operations UI
system-prompt UI
protocol logging
Nuntius commands
Pontis heartbeat
Vestigare prompt provenance
```

Again, these aren't necessarily wrong.

But much of this grew because Moderari needed to make itself observable before Lumen had today's overall architecture.

Now we have:

```text
Servire
Nuntius
Vestigare
Pontis
Praebere
Rogare
Fiducia
Repetere
```

So there is significant potential duplication of authority.

---

# 4. Configuration is another clue

Your point about Moderari having more configuration possibilities than almost anything else is important.

Configuration often reveals hidden responsibilities.

The current configuration essentially says Moderari needs to know about:

```text
HTTP/server
MongoDB
provider
model profiles
context
checkpointing
prompt policy
tool behaviour
continuation
logging
protocol logging
Nuntius
Pontis
timeouts
UI
session persistence
```

That is a lot of worlds for one service to inhabit.

Minimal Moderari's configuration could potentially become closer to:

```yaml
moderari:
  host: 0.0.0.0
  port: 11436

model_profiles:
  path: profiles

nuntius:
  endpoint: ...

logging:
  ...
```

Potentially not even provider configuration.

Because today:

```text
Servire
   │
   ▼
Praebere
   │
   │ provider/model authority
   ▼
Pontis/session
   │
   │ selected provider/model
   ▼
Moderari
```

Moderari should arguably be **told what execution target applies to this session/request**, rather than having its own concept of a globally configured provider.

That becomes particularly important for the next thing already planned:

> **each session can have its own model.**

And later:

> **multiple providers, protocols and clients.**

---

# 5. That changes the future Moderari shape substantially

Instead of:

```text
                  Moderari
                     │
                     │ configured Ollama
                     ▼
                   Ollama
```

we are moving towards:

```text
                       Moderari
                          │
              ┌───────────┼────────────┐
              │           │            │
              ▼           ▼            ▼
          Provider A  Provider B   Provider C
              │           │            │
              ▼           ▼            ▼
          Model A      Model B      Model C
```

with the execution context saying something like:

```text
session
provider
model
protocol
model profile
tool capability
context characteristics
```

That means Moderari increasingly becomes an **adapter pipeline**, not a configured proxy.

Something conceptually like:

```text
                 Incoming interaction
                         │
                         ▼
                Canonical Lumen form
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       Context policy          Prompt policy
             │                       │
             └───────────┬───────────┘
                         ▼
                  Model adapter
                         │
                         ▼
                Provider adapter
                         │
                         ▼
                      Model
                         │
                         ▼
                Provider adapter
                         │
                         ▼
                  Model adapter
                         │
                         ▼
                Canonical Lumen form
```

That's a much cleaner architecture.

---

# 6. And this gives us a useful test for every existing feature

For each current Moderari capability, I suggest we ask exactly five questions:

1. **Why does this capability exist?**
2. **Is the original problem still present in today's Lumen?**
3. **Which service now has the authoritative information needed to solve it?**
4. **Does it alter the model-facing interaction?**
5. **If we removed it from Moderari, where would it naturally belong?**

Question 4 is particularly powerful.

If the answer is:

> “Yes, this determines what goes into or comes out of the model.”

there is a strong argument for Moderari.

If the answer is:

> “No, it manages the execution/session/system around the model.”

then we should be suspicious of it being in Moderari.

---

# 7. Rebuild our understanding in layers

Rather than going through the existing files one by one, use this sequence:

```text
LAYER 0
Minimal transport
Pontis → Moderari → provider → Moderari → Pontis

LAYER 1
Model/provider adaptation
Canonical request ↔ model-specific representation

LAYER 2
Effective model context
System prompt + messages + tools + model profile

LAYER 3
Context limits
Measurement + checkpoint/continuity

LAYER 4
Execution assistance
Recovery + read workflow + tool suppression

LAYER 5
Persistence
Sessions + checkpoints + results

LAYER 6
Evidence
Protocol + effective prompt + trace correlation

LAYER 7
Operations
State + heartbeat + Nuntius + UI + commands
```

At **each layer**, decide whether it stays, moves, changes or disappears.

That should get us to a Moderari architecture designed for **Lumen as it exists in late 2026**, rather than continuing to extend the original Moderari.

I think **Layer 0 is where we should go next**: define precisely what the absolutely minimal Moderari would look like—its API contract, what Pontis gives it, what it obtains from Praebere/Nuntius, what it sends to a provider, and what it returns—with *all checkpointing, Mongo, recovery, UI, system-prompt manipulation and other extras switched off*.

Then we can add Layer 1 and see exactly why each additional piece exists.

The implementation can also naturally reflect those boundaries with separate components rather than continuing to accumulate responsibilities in `proxy.py`.

That exercise may also reveal that **“minimal Moderari” no longer needs MongoDB at all**, which would be a significant architectural simplification.