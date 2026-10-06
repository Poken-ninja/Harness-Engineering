# KNOWLEDGE_MAP.md

## Purpose

This file is the entry point for the Project's reusable operating knowledge.

Use it to determine:

- which source to consult for a given question;
- which source is authoritative when two sources disagree;
- which information is universal guidance versus a project-specific decision;
- where changing execution state belongs;
- when external or current information must be consulted instead of relying on static Project documents.

This file is a map, not a complete operating manual.

---

## 1. Source Classes

The Project uses four distinct classes of knowledge.

### A. Governing instructions

**Source:** Project instructions

**Contains:**
- the role of the Harness and Loop Architect;
- required behavior and exclusions;
- required deliverable structure;
- quality expectations;
- restrictions such as not implementing the application.

**Consult when:**
- determining what this Project is allowed to do;
- resolving ambiguity about the Project's role;
- deciding the required form of an answer.

**Authority:** Highest within this Project's reusable documents.

**Rule:** Supporting documents may make the instructions more operational, but must not silently override them.

---

### B. Reusable engineering rules

These documents define how future ideas are analyzed.

#### `HARNESS_RULES.md`

**Contains:**
- context-selection rules;
- source authority and freshness rules;
- environment and startup readiness;
- tool and permission boundaries;
- scope and WIP controls;
- persistent-state requirements;
- checkpoint and handoff requirements.

**Consult when asking:**
- What information must the builder have?
- Which source should be trusted?
- Is the environment ready for implementation?
- What may the builder modify or access?
- How should scope be bounded?
- What must survive between sessions?

**Does not contain:**
- the detailed retry algorithm for an individual task;
- the full output format for analyzing a new idea.

Those belong in `LOOP_RULES.md` and `EXECUTION_CONTRACT_TEMPLATE.md`.

---

#### `LOOP_RULES.md`

**Contains:**
- task lifecycle and allowed state transitions;
- task entry requirements;
- action → verification → repair control flow;
- failure classification;
- retry/resource budgets;
- blocked and escalation conditions;
- stop conditions;
- checkpoint, rollback, and recovery behavior;
- rules for invalidating previous verification.

**Consult when asking:**
- May this task start?
- What happens after an implementation attempt?
- What happens when verification fails?
- How many retries are allowed?
- When should the task stop, block, roll back, or escalate?
- What evidence permits a state transition?

**Does not contain:**
- project-specific task details;
- general source-selection rules.

---

#### `EXECUTION_CONTRACT_TEMPLATE.md`

**Contains:**
- the standard structure used to transform a future idea, prompt, or plan into a builder-ready execution contract;
- placeholders for objective, facts, assumptions, context, readiness, bounded tasks, loops, verification, persistence, and unresolved gaps.

**Consult when:**
- the user supplies a new idea;
- the user supplies an existing prompt or plan to improve;
- a fresh execution specification must be produced.

**Rule:** The template defines what must be specified. It does not prove that any mechanism exists or that any task has been executed.

---

#### `QUALITY_RUBRIC.md`

**Contains:**
- checks for contract completeness;
- checks for contradictions and hidden scope expansion;
- verifiability checks;
- recoverability checks;
- enforcement/evidence checks;
- complexity and coordination checks.

**Consult when:**
- reviewing a draft execution contract before handing it to a builder;
- revising an existing contract;
- deciding whether additional harness or graph complexity is justified.

**Rule:** A contract should not be treated as ready merely because every section is filled in. It must satisfy the rubric materially.

---

### C. Project-specific facts and decisions

**Source:** Future project-specific artifacts referenced by an execution contract.

Examples may include:

```text
[PROJECT_BRIEF]
[ARCHITECTURE_DOCUMENT]
[REPOSITORY_PATH]
[API_SPECIFICATION]
[DESIGN_SOURCE]
[SECURITY_POLICY]
[CURRENT_TASK_STATE]
[VERIFICATION_RESULTS]
[DECISION_LOG]
```

These are placeholders until an actual project exists.

**Contains:**
- facts about the specific product;
- repository structure;
- architecture decisions;
- requirements;
- constraints;
- current execution state;
- actual verification evidence.

**Consult when:**
- a reusable rule needs concrete project information;
- determining actual implementation scope;
- determining whether work has really passed verification.

**Authority:** Authoritative only for the domain explicitly assigned to the source.

Example:

- an API specification may be authoritative for API behavior;
- the repository may be authoritative for current code;
- a production monitoring system may be authoritative for current runtime health.

No source becomes globally authoritative merely because it is stored in the repository.

---

### D. External or live sources

Examples:

- official product documentation;
- live APIs;
- CI results;
- issue trackers;
- production monitoring;
- vendor systems;
- applicable laws or standards;
- user-provided current facts.

**Consult when:**
- the answer depends on information that may change independently of the Project documents;
- a Project document explicitly delegates authority to an external source;
- actual execution evidence must be obtained from a running system.

**Rule:** Static documentation must not be used as proof of current runtime state when a live source is authoritative.

---

## 2. Authority Order

Do not use one universal hierarchy for every fact.

Resolve conflicts by **domain authority**.

Use this sequence:

1. Identify the disputed claim.
2. Identify which source is designated authoritative for that type of claim.
3. Check whether that source is current enough for the decision.
4. If two authoritative sources conflict, record the conflict as unresolved instead of silently choosing one.
5. If no authority has been designated, treat the claim as an assumption until the execution contract assigns one.

### Example

Suppose:

- `docs/api.md` says the endpoint is `/v1/users`;
- the deployed API reports `/v2/users`;
- the execution contract says the live staging API is authoritative for currently deployed behavior.

Then:

- `/v2/users` is the current runtime fact;
- the documentation is stale;
- the discrepancy should be recorded and repaired.

Do not pretend that storing `docs/api.md` in the repository makes it more authoritative than observed runtime behavior.

---

## 3. Knowledge Categories

When analyzing a new idea, classify information before using it.

### Fact

Directly supplied or supported by an identified authoritative source.

Format:

```text
FACT:
[statement]

SOURCE:
[source]
```

### Assumption

A reversible belief used because required information is unavailable.

Format:

```text
ASSUMPTION:
[statement]

WHY NEEDED:
[reason]

INVALIDATED IF:
[evidence]
```

### Decision

A choice made between viable alternatives.

Format:

```text
DECISION:
[choice]

RATIONALE:
[reason]

OWNER:
[decision authority]
```

### Unknown

Information that materially affects the design but cannot currently be established.

Format:

```text
UNKNOWN:
[question]

IMPACT:
[what changes depending on the answer]
```

Do not convert unknowns into facts merely to make a contract look complete.

---

## 4. Principle, Default, and Project Decision

Reusable guidance must be labeled according to its strength.

### Principle

A general engineering property the Project intends to preserve.

Example:

> Completion must be supported by observable evidence.

Principles should rarely depend on a particular tool.

---

### Default

The normal choice when project-specific evidence does not justify another option.

Example:

> Default to one active implementation task.

A default may be overridden, but the execution contract must state why.

---

### Project-dependent decision

A value that cannot be selected correctly until the actual project is known.

Examples:

```text
[RETRY_LIMIT]
[VERIFICATION_COMMAND]
[STATE_STORAGE_LOCATION]
[ALLOWED_TOOLS]
[PARALLEL_WORK_LIMIT]
[REQUIRED_REVIEWER]
```

Do not turn these placeholders into universal rules.

---

## 5. Evidence Levels

Keep specifications separate from actual execution evidence.

### Specified

A requirement or mechanism has been written down.

Example:

> CI must run the end-to-end login test.

This proves only that the contract contains the requirement.

### Mechanized

A real mechanism exists that can enforce or evaluate the requirement.

Example:

> The CI workflow contains a job that executes the login test.

This proves the mechanism exists, not that it passed.

### Verified

Evidence shows the mechanism actually succeeded for the relevant version.

Example:

```text
Commit: [SHA]
Check: login-e2e
Result: PASS
Timestamp: [TIME]
Evidence: [CI RUN / LOG]
```

Only this level supports a claim that the behavior was verified.

**Project addition:** This three-level distinction is mandatory whenever a contract describes automation, verification, persistent state, isolation, or independent review. It prevents documents from being mistaken for functioning infrastructure.

---

## 6. Freshness and Invalidation

A source may be authoritative and still be stale.

For consequential information, an execution contract should identify:

```text
SOURCE:
[authoritative source]

FRESHNESS REQUIREMENT:
[when it must be rechecked]

INVALIDATION EVENT:
[change that makes previous evidence unreliable]
```

Typical invalidation events include:

- relevant code changed;
- dependencies changed;
- acceptance criteria changed;
- environment configuration changed;
- an upstream contract changed;
- verification tooling changed;
- authoritative external data changed.

Previously passing evidence must not automatically be treated as current after an invalidation event.

---

## 7. How to Use This Knowledge Set

For every new idea or prompt:

1. Read the Project instructions.
2. Use `KNOWLEDGE_MAP.md` to identify the relevant sources.
3. Consult `HARNESS_RULES.md` for context, readiness, scope, tools, state, and handoff design.
4. Consult `LOOP_RULES.md` for task execution, verification, failure, retry, stop, and recovery design.
5. Fill `EXECUTION_CONTRACT_TEMPLATE.md` using project-specific facts and explicit placeholders for unknowns.
6. Review the result using `QUALITY_RUBRIC.md`.
7. Repair contradictions, unverifiable claims, missing dependencies, unbounded retries, unavailable mechanisms, and unnecessary complexity before handoff.
8. Keep actual execution status and verification evidence outside the reusable rule documents.

---

## 8. What Must Not Be Stored Here

Do not turn this map into a progress log or project encyclopedia.

Do not store:

- current implementation status;
- transient failure logs;
- project-specific credentials;
- temporary research notes;
- feature-by-feature execution evidence;
- long procedural instructions already owned by another source.

Those belong in project-specific state or evidence artifacts defined by the relevant execution contract.

---

## 9. Conflict Rule

When sources disagree:

> Do not reconcile the conflict by inventing an answer.

Record:

```text
CONFLICT:
[source A] says [...]
[source B] says [...]

DOMAIN:
[what the disagreement affects]

CURRENT AUTHORITY:
[source, if already defined]

ACTION:
[use authoritative source / request decision / refresh stale source]

STATUS:
[resolved | unresolved]
```

An unresolved consequential conflict blocks any task whose correctness depends on resolving it.

---

## 10. Minimal Entry Instruction

When entering this Project with a new idea:

> Use the Project instructions as governing policy. Use this map to select the smallest relevant set of reusable rules and project-specific sources. Treat facts, assumptions, decisions, and unknowns separately. Do not claim that a written rule is enforced or that a behavior is verified without evidence from the mechanism responsible for it. Produce the execution contract, then evaluate it with the quality rubric before handing it to a builder.
