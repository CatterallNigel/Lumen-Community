# Lumen Assess Development Corpus — Initial Task Set

**Status:** Initial baseline  
**Purpose:** Create a small, intentional corpus of traced Lumen runs that can support development of Lumen Assess.  
**Initial approach:** Start with tasks that have objectively verifiable answers, then progress toward tasks where quality, completeness, assumptions, tool use, and behavioural variation become more significant.

## Experimental intent

The first corpus should not simply contain many unrelated prompts. The aim is to create repeated, comparable runs so that Assess can eventually distinguish:

- answer correctness;
- reasoning-path variation;
- instruction compliance;
- assumption formation;
- tool selection and tool avoidance;
- completeness and capability utilisation;
- behavioural stability across repeated runs;
- sensitivity to prompt wording and context.

Where practical, each task should be run repeatedly under the same conditions before varying those conditions.

An important characteristic of the current stack is that **Qwen may choose to use tools even for apparently simple tasks**, including arithmetic via Bash or other available tools. Tool use should therefore be treated as part of the observed behaviour rather than removed from the experiment. For some tasks, whether the model invokes a tool is itself useful evidence.

---

## Initial task set

| Order | Class | Task | What it examines |
|---|---|---|---|
| 1 | Deterministic arithmetic | **Calculate 17 × 23. Give the numerical answer and explain the calculation.** | Objective correctness; stability of the final answer; variation in reasoning path; whether Qwen chooses to invoke a tool for a simple deterministic calculation. |
| 2 | Constraint following | **Give exactly five prime numbers between 50 and 100, in ascending numerical order, with no additional text.** | Objective correctness; exact instruction compliance; formatting discipline; whether unnecessary explanation or tool use is introduced. |
| 3 | Simple reasoning | **A shop sells an item for £120. It reduces the price by 20%, then later increases the reduced price by 20%. Is the final price £120? Calculate the final price and explain why.** | Objective answer correctness; multi-step reasoning; recognition that percentage decrease and increase are applied to different bases; explanation quality; possible calculator/tool use. The correct final price is £115.20. |
| 4 | Diagnostic | **The following Python function is intended to return the average of a list of numbers, but it fails for an empty list:** `def average(values): return sum(values) / len(values)` **Identify the defect and provide a corrected implementation that handles an empty list safely. Explain the design choice you make for the empty-list case.** | Fault localisation; correctness of proposed fix; handling of edge cases; explicit design assumptions; possible use of coding or execution tools; variation in how the empty case is defined. |
| 5 | Trade-off | **A web application stores customer accounts, orders, payments, and transactional relationships between those records. It also stores flexible user-generated profile metadata whose fields may vary frequently between users. Explain when PostgreSQL would be preferable to MongoDB for this system, when MongoDB would be preferable, and whether you would choose one database or a mixed architecture. Justify the decision.** | Trade-off reasoning; recognition of relational integrity versus schema flexibility; quality of justification; assumptions; architectural synthesis; answer stability where there is no single mandatory architecture. |
| 6 | Planning | **A small REST API currently runs on a single virtual machine. Design a migration plan to move it into containers without planned downtime. The service uses a relational database and must remain available during the migration. Describe the migration stages, validation points, rollback approach, and cut-over strategy.** | Planning depth; sequencing; risk identification; rollback awareness; completeness; operational reasoning; variation in migration strategies. |
| 7 | Synthesis | **You are given a small internal service that receives HTTP requests, performs business validation, writes records to a database, and publishes an event after each successful transaction. Propose an implementation architecture that separates responsibilities cleanly. Describe the main components, request flow, failure handling, observability, and how you would test the design.** | Architectural decomposition; synthesis of multiple concerns; completeness; capability utilisation; handling of failure and observability; variation in structure and emphasis. |
| 8 | Bounded-resource reasoning | **You have two software engineers and four weeks to improve the reliability of a production API that suffers from intermittent failures, poor diagnostics, and occasional slow responses. You cannot rewrite the system. Propose a prioritised four-week plan, explain what you would deliberately not attempt, and define how you would decide whether the work had been successful.** | Satisficing under constraints; prioritisation; ability to reject low-value work; bounded-resource decision quality; measurable success criteria; consistency of priorities across runs. |
| 9 | Ambiguous problem | **A company says that customer waiting times are too long and asks you to reduce them. You are given no further information. Explain how you would approach the problem before recommending a solution, what information you would seek, what assumptions you would avoid making, and what kinds of interventions might eventually be appropriate.** | Assumption management; clarification strategy; resistance to premature solutioning; breadth of possible approaches; behavioural variation under ambiguity. |
| 10 | Open-ended assurance design | **Design an audit mechanism for an AI system whose outputs are probabilistic and may differ across repeated executions of the same task. The mechanism should help determine whether behavioural differences are harmless variation or meaningful divergence. Describe what evidence should be captured, how repeated runs should be compared, and how confidence in the system should develop over time.** | Open-ended reasoning; conceptual depth; treatment of nondeterminism; evidence design; distinction between variation and harmful divergence; potential overlap with Lumen's own assurance concepts; high expected behavioural variation. |

---

## Recommended execution order

Development should begin with the first four tasks because they provide comparatively strong external reference points:

1. deterministic arithmetic;
2. exact constraint following;
3. simple percentage reasoning;
4. code diagnosis.

These allow Assess development to begin with signals that are comparatively easy to verify before moving into tasks where evaluation is increasingly about quality, sufficiency, assumptions, trade-offs, or decision structure.

The later tasks should then be introduced in the order shown above, moving progressively from constrained reasoning toward open-ended behaviour.

## Repetition strategy

For an initial baseline, each task should be executed multiple times under otherwise unchanged conditions. A starting target of **10–20 clean-session runs per task** should be sufficient to begin exposing behavioural patterns without creating an unnecessarily large corpus.

Each run should preserve, where available:

- task identifier;
- exact user prompt;
- model and model version;
- system prompt or system-prompt version;
- tool inventory exposed to the model;
- tools actually invoked;
- tool arguments and returned evidence where appropriate;
- final answer;
- trace identifier;
- replay identifiers for subsequent reruns;
- timestamps;
- relevant runtime/model parameters;
- whether the session began with clean context;
- any divergence detected during Replay.

The first baseline should hold as many of these variables constant as possible.

---

## Tool use is part of the experiment

Qwen currently has access to tools and may invoke them even for tasks that could be solved internally, including simple arithmetic.

This should **not initially be suppressed**.

For Assess, several outcomes may ultimately matter independently:

- Was the answer correct?
- Was a tool used?
- Was the selected tool appropriate?
- Was tool use necessary?
- Did tool use improve reliability?
- Did repeated runs choose different tools or alternate between tool use and internal reasoning?
- Did equivalent answers result from materially different behavioural paths?

This makes apparently simple tasks especially useful. For example, the arithmetic task can be objectively correct in every run while still producing variation in whether Bash, another calculator-like mechanism, or no tool is used.

A later experiment can deliberately compare **tool-enabled** and **tool-restricted** conditions, but the first corpus should capture the stack behaving naturally.

---

## Moderari system prompt as an experimental variable

Moderari currently creates and injects the system prompt. This means the system prompt is part of the model's execution environment and may materially influence:

- reasoning style;
- verbosity;
- willingness to use tools;
- tool-selection behaviour;
- compliance with constraints;
- assumption formation;
- answer structure;
- caution or confidence;
- capability utilisation.

For the initial corpus, the Moderari system prompt should therefore be treated as a **controlled variable** and kept unchanged.

It should also be recorded, or at minimum versioned or hashed, alongside each experiment so that later comparisons remain interpretable.

Once a baseline exists, system-prompt optimisation can become a deliberate second-order experiment. The same task corpus can be rerun with controlled changes to the Moderari system prompt and compared against the baseline.

This would allow Lumen to investigate not merely whether a prompt produces a different answer, but whether it produces:

- more stable behaviour;
- better instruction compliance;
- more appropriate tool use;
- higher-quality reasoning;
- fewer unnecessary actions;
- improved capability utilisation;
- less harmful divergence across repeated runs.

That makes system-prompt optimisation a natural future use case for **Trace + Replay + Fiducia + Assess** rather than a separate prompt-engineering exercise.

---

## Initial corpus principle

The first objective is not to prove that the model is intelligent or to rank it.

The objective is to establish enough high-quality evidence that Assess can begin answering a more useful question:

> **When the same or equivalent task is performed repeatedly, what changes, what remains stable, and which differences actually matter?**

