# HARNESS_RULES.md

## Purpose

This document defines the reusable harness rules used when turning a future idea, prompt, or plan into an execution contract.

It governs:

- context selection;
- source authority;
- environment readiness;
- tool and permission boundaries;
- scope control;
- persistent state;
- checkpoints and handoffs.

It does **not** define detailed retry loops, task-state transition logic, or failure-repair budgets. Those belong in `LOOP_RULES.md`.

---

# 1. Rule Strength

Rules in this document use three levels.

## Principle

A system property that should hold across projects unless the Project instructions explicitly override it.

## Default

The preferred choice when project-specific evidence does not justify something else.

Defaults may be overridden with a recorded rationale.

## Project-dependent choice

A value that must be selected for the actual project.

Examples:

```text
[AUTHORITATIVE_REQUIREMENTS_SOURCE]
[STATE_STORAGE_LOCATION]
[ALLOWED_TOOLS]
[STARTUP_COMMAND]
[VERIFICATION_ENVIRONMENT]
[PARALLEL_WIP_LIMIT]
```

Do not silently convert defaults into universal requirements.

---

# 2. Context Selection

## HR-01 — Use the smallest sufficient context

**Strength:** Principle

**Requirement**

Provide the builder with the smallest set of sources sufficient to perform the active task correctly.

Do not preload unrelated project knowledge merely because it exists.

**Failure mode prevented**

- context dilution;
- irrelevant instructions competing with task reasoning;
- stale or contradictory information entering the working set;
- excessive discovery cost.

**Applies when**

Preparing context for any implementation, verification, review, or recovery task.

**Enforcement mechanism**

The execution contract must list:

```text
REQUIRED_CONTEXT:
- [source]
- [source]

OPTIONAL_ON_DEMAND_CONTEXT:
- [source + trigger]

EXCLUDED_CONTEXT:
- [source + reason, if consequential]
```

**Required evidence**

A reviewer can map every required source to either:

- an acceptance criterion;
- a constraint;
- a dependency;
- a verification step;
- a recovery need.

**Response to violation**

Remove irrelevant sources or explain why each retained source is necessary.

Do not solve low-quality context by adding more context.

---

## HR-02 — Route from entry documents to deeper sources

**Strength:** Default

**Requirement**

Use short entry-point instructions that direct the builder to deeper material only when needed.

**Failure mode prevented**

- giant instruction files;
- low signal-to-noise ratio;
- important constraints becoming buried.

**Applies when**

The project contains multiple domains, subsystems, or operating procedures.

**Enforcement mechanism**

The execution contract should identify an entry source such as:

```text
[ENTRY_INSTRUCTIONS]
```

and topic-specific sources such as:

```text
[ARCHITECTURE_RULES]
[TESTING_RULES]
[DATABASE_RULES]
[FRONTEND_RULES]
```

**Required evidence**

A fresh builder can identify the correct deeper source without searching the whole repository.

**Response to violation**

Split overloaded entry material and add routing instructions.

**Note**

Suggested file lengths from course material are teaching defaults, not compliance thresholds.

---

## HR-03 — Separate durable context from transient context

**Strength:** Principle

**Requirement**

Distinguish:

- durable project knowledge;
- current execution state;
- transient task-local observations.

Do not mix all three into a single artifact.

**Failure mode prevented**

- stale progress data appearing as permanent architecture guidance;
- temporary debugging observations becoming policy;
- durable rules being overwritten by execution chatter.

**Applies when**

Designing project state or documentation structure.

**Enforcement mechanism**

Assign each piece of information to one of:

```text
DURABLE_KNOWLEDGE
CURRENT_STATE
TASK_LOCAL_CONTEXT
```

**Required evidence**

The execution contract names where each category is stored.

**Response to violation**

Move information to the appropriate artifact before relying on it operationally.

---

# 3. Source Authority

## HR-04 — Authority is domain-specific

**Strength:** Principle

**Requirement**

Assign authority by claim type rather than declaring one source globally authoritative.

Example:

```text
Requirements           → [PRODUCT_SPEC]
Current code            → [REPOSITORY]
Deployed behavior       → [LIVE_ENVIRONMENT]
API contract            → [API_SPEC]
Current CI status       → [CI_SYSTEM]
Legal requirement       → [OFFICIAL_LEGAL_SOURCE]
```

**Failure mode prevented**

- trusting stale documentation over observed runtime behavior;
- treating repository text as authoritative for external facts;
- resolving conflicts arbitrarily.

**Applies when**

More than one source can speak to the same project.

**Enforcement mechanism**

Every execution contract must contain a source-of-truth map for consequential domains.

**Required evidence**

For every critical claim, the evaluator can identify its governing source.

**Response to violation**

Mark the affected fact as unresolved until authority is assigned.

---

## HR-05 — Freshness must be explicit where facts can drift

**Strength:** Principle

**Requirement**

For sources whose contents may change independently, define when they must be refreshed.

**Failure mode prevented**

- decisions based on stale APIs;
- outdated runtime assumptions;
- obsolete dependency or environment information.

**Applies when**

A source represents:

- live systems;
- external services;
- mutable specifications;
- changing policies;
- deployment state;
- dependency versions.

**Enforcement mechanism**

Record:

```text
SOURCE:
[...]
FRESHNESS_RULE:
[...]
INVALIDATION_EVENT:
[...]
```

**Required evidence**

The evidence record includes a timestamp, version, revision, or equivalent freshness marker when material.

**Response to violation**

Refresh the source before continuing or mark the dependent task blocked.

---

## HR-06 — Conflicts must remain visible

**Strength:** Principle

**Requirement**

Do not silently merge contradictory sources.

**Failure mode prevented**

- fabricated certainty;
- implementation against the wrong contract;
- hidden stale documentation.

**Applies when**

Two relevant sources disagree on a consequential fact.

**Enforcement mechanism**

Record the conflict using the conflict format from `KNOWLEDGE_MAP.md`.

**Required evidence**

Either:

- the designated authority resolves the conflict; or
- an explicit decision resolves it.

**Response to violation**

Block dependent implementation if the conflict can materially change behavior.

---

# 4. Environment Readiness

## HR-07 — Initialization is distinct from implementation

**Strength:** Principle

**Requirement**

Do not begin feature implementation until the minimum operating environment required for that feature is established.

Initialization work must be identified separately.

**Failure mode prevented**

- mixing setup and feature work;
- unclear failures caused by environment versus code;
- spending implementation budget discovering basic project operation.

**Applies when**

A builder cannot yet demonstrate the required startup, test, or inspection capability.

**Enforcement mechanism**

The execution contract must include a readiness section before implementation tasks.

**Required evidence**

The required readiness checks have observable results.

**Response to violation**

Reclassify the work as initialization and do not mark the feature task active.

---

## HR-08 — Startup readiness requires demonstrated capabilities

**Strength:** Default

A project is implementation-ready for a task when the builder can demonstrate the capabilities relevant to that task.

Typical readiness dimensions:

```text
START:
Can the relevant system start?

VERIFY:
Can the required checks execute?

OBSERVE:
Can failures be inspected through logs/output/state?

RESUME:
Can another session discover current state and next action?
```

Not every project requires identical commands.

**Failure mode prevented**

- coding in an environment that cannot verify the result;
- hidden runtime failures;
- unrecoverable session boundaries.

**Applies when**

Beginning a new project, entering an unfamiliar repository, or recovering from major environment change.

**Enforcement mechanism**

Project-specific startup checks.

**Required evidence**

Actual command results, environment observations, or tool responses.

A written checklist alone is not evidence.

**Response to violation**

Mark readiness as incomplete and create a bounded initialization task.

---

## HR-09 — Do not invent unavailable capabilities

**Strength:** Principle

**Requirement**

Tooling, permissions, environments, test systems, reviewers, and automation mechanisms must be either:

- confirmed available;
- explicitly required but unverified;
- explicitly unavailable.

**Failure mode prevented**

Execution contracts that depend on fictional infrastructure.

**Applies when**

Specifying any tool-assisted or automated mechanism.

**Enforcement mechanism**

Maintain a readiness table:

```text
CAPABILITY | REQUIRED | STATUS | EVIDENCE
```

Allowed statuses:

```text
available
unverified
unavailable
not_required
```

**Required evidence**

A real source establishing availability.

**Response to violation**

Replace the assumption with an unresolved dependency or design an alternative that uses available capabilities.

---

# 5. Tool and Permission Boundaries

## HR-10 — Grant only tools required by the active task

**Strength:** Principle

**Requirement**

A task contract must specify permitted tools or tool classes and prohibit unnecessary privileged access.

**Failure mode prevented**

- accidental scope expansion;
- destructive actions unrelated to the task;
- unnecessary security exposure;
- difficult-to-audit execution.

**Applies when**

The builder can interact with repositories, shells, deployment systems, external services, production data, or other mutable systems.

**Enforcement mechanism**

Specify:

```text
PERMITTED:
- [tool/action]

PROHIBITED:
- [tool/action]

REQUIRES_APPROVAL:
- [tool/action]
```

**Required evidence**

Actual permission configuration where enforcement exists.

If only written instructions exist, label the rule **advisory**.

**Response to violation**

Stop execution if the action may create material risk.

Record any performed out-of-bound action as an incident requiring review.

---

## HR-11 — Separate read, write, and destructive permissions

**Strength:** Default

**Requirement**

Treat these as distinct capability levels:

```text
READ
WRITE
DESTRUCTIVE / IRREVERSIBLE
```

Do not infer permission for one from permission for another.

**Failure mode prevented**

A tool that can inspect a system being assumed safe to modify or delete from it.

**Applies when**

External or mutable systems are involved.

**Enforcement mechanism**

Explicit permission matrix.

**Required evidence**

Tool configuration, access policy, or confirmed capability boundaries.

**Response to violation**

Downgrade the task to the highest confirmed permission level.

---

## HR-12 — High-impact actions need explicit gates

**Strength:** Principle

**Requirement**

Actions with material irreversible or external consequences require an explicit approval or automated gate appropriate to the risk.

Examples may include:

- production deployment;
- destructive migration;
- deletion;
- publishing;
- merging to protected branches;
- sending external communications;
- spending money;
- changing access control.

**Failure mode prevented**

An agent converting a local reasoning error into a real-world irreversible action.

**Applies when**

The task includes high-impact operations.

**Enforcement mechanism**

Project-dependent gate such as:

```text
human approval
protected environment
CI requirement
change-management system
transaction preview
dry run
```

**Required evidence**

The gate actually occurred.

**Response to violation**

Do not perform the action.

If already performed, stop further changes and enter recovery/escalation.

---

# 6. Scope Control

## HR-13 — Every implementation task has a bounded contract

**Strength:** Principle

**Requirement**

No implementation task begins without:

```text
TASK_ID
OBJECTIVE
IN_SCOPE
OUT_OF_SCOPE
DEPENDENCIES
ACCEPTANCE_CRITERIA
VERIFICATION_METHOD
STATE
```

**Failure mode prevented**

- scope creep;
- invisible side work;
- inability to distinguish partial from complete work.

**Applies when**

Any implementation or project-modifying task is proposed.

**Enforcement mechanism**

Task contract required by `EXECUTION_CONTRACT_TEMPLATE.md`.

**Required evidence**

Each acceptance criterion can be mapped to a verification method.

**Response to violation**

Do not activate the task until the missing boundary is defined.

---

## HR-14 — Default to one active implementation task

**Strength:** Default

**Requirement**

Use WIP=1 unless parallel execution has an explicit justification.

This is a scope-control default, not a universal law.

**Failure mode prevented**

- many partially completed changes;
- cross-task interference;
- diluted verification effort;
- difficult recovery.

**Applies when**

Tasks share files, dependencies, runtime state, reviewer capacity, or uncertain boundaries.

**Enforcement mechanism**

State storage permits only the configured active-task limit.

Where no mechanism exists, this rule is advisory and must be labeled as such.

**Required evidence**

Current task state shows no excess active implementation work.

**Response to violation**

Stop activating new tasks.

Finish, block, or explicitly de-activate existing work before proceeding.

---

## HR-15 — Parallel work requires justification and isolation

**Strength:** Principle

**Requirement**

Parallel execution is permitted only when all of the following are defined:

```text
INDEPENDENT_SCOPE
DEPENDENCY_RELATIONSHIP
STATE_OWNERSHIP
COLLISION_AVOIDANCE
INTEGRATION_POINT
VERIFICATION_AFTER_INTEGRATION
```

**Failure mode prevented**

- conflicting edits;
- duplicate work;
- hidden dependency races;
- individually passing branches that fail when combined.

**Applies when**

More than one implementation task is active.

**Enforcement mechanism**

Project-dependent isolation mechanism such as separate worktrees, branches, sandboxes, or non-overlapping resources.

**Required evidence**

Isolation exists and integrated behavior is subsequently verified.

**Response to violation**

Return to serial execution.

---

## HR-16 — Incidental discoveries do not expand active scope automatically

**Strength:** Principle

**Requirement**

When the builder discovers unrelated defects or improvements, record them without implementing them unless they block the active task.

**Failure mode prevented**

“While I am here” refactoring and uncontrolled feature expansion.

**Applies when**

A new issue is discovered during active work.

**Enforcement mechanism**

Record as:

```text
DISCOVERED_ITEM:
[description]

RELATION_TO_ACTIVE_TASK:
blocking | non_blocking

ROUTE:
address_now | queue | escalate
```

Only blocking issues may enter the active task, and only with documented rationale.

**Required evidence**

Task diff and handoff show no unexplained unrelated work.

**Response to violation**

Revert or separate unrelated changes unless they are explicitly accepted into scope.

---

# 7. Persistent State

## HR-17 — Conversation history is not durable project state

**Strength:** Principle

**Requirement**

Any information required to resume work after a fresh session must exist in an external durable artifact.

**Failure mode prevented**

- lost decisions;
- repeated investigation;
- contradictory next steps;
- false reliance on hidden conversational memory.

**Applies when**

Work may span more than one session or agent.

**Enforcement mechanism**

Project-specific persistent state storage.

Example placeholders:

```text
[CURRENT_TASK_STATE]
[PROGRESS_LOG]
[DECISION_LOG]
[VERIFICATION_RECORD]
```

**Required evidence**

A fresh session can reconstruct the current task without inaccessible prior conversation.

**Response to violation**

Create or repair the missing state artifact before handoff.

---

## HR-18 — Separate specification state from execution state

**Strength:** Principle

**Requirement**

Do not store planned behavior and actual completion evidence as if they were the same thing.

Example:

```text
SPECIFICATION:
"Login E2E test must pass."

EXECUTION STATE:
"verification pending"

EVIDENCE:
none
```

**Failure mode prevented**

Requirements being mistaken for achieved results.

**Applies when**

Recording task status or verification.

**Enforcement mechanism**

Separate fields or artifacts for:

```text
required
current_state
evidence
```

**Required evidence**

Status claims point to evidence rather than repeating the specification.

**Response to violation**

Downgrade the status to unverified.

---

## HR-19 — Decisions that constrain future work must be durable

**Strength:** Principle

**Requirement**

Record decisions that materially constrain architecture, interfaces, security, scope, or verification.

Do not require future sessions to infer them from code or conversation.

**Failure mode prevented**

- repeated reopening of settled questions;
- contradictory implementations;
- hidden rationale.

**Applies when**

A decision affects future task design.

**Enforcement mechanism**

Use:

```text
DECISION_ID
DECISION
RATIONALE
AUTHORITY
DATE_OR_VERSION
AFFECTED_SCOPE
INVALIDATION_CONDITION
```

**Required evidence**

The decision exists in the designated durable source.

**Response to violation**

Treat the issue as undecided until the decision is reconstructed or re-made.

---

## HR-20 — State writes must be attributable

**Strength:** Project addition

**Requirement**

Consequential state transitions and evidence updates should identify what caused them.

**Failure mode prevented**

A task mysteriously becoming “done” with no way to determine why.

**Applies when**

Updating task state, verification status, or major decisions.

**Enforcement mechanism**

Record:

```text
CHANGE:
[...]
CAUSE:
[verification / decision / dependency change / recovery]
EVIDENCE:
[reference]
```

**Required evidence**

The state transition can be traced to a recorded event.

**Response to violation**

Do not trust the transition until reconstructed.

---

# 8. Verification Evidence and Invalidation

## HR-21 — Written requirements do not count as enforcement

**Strength:** Principle

**Requirement**

For consequential controls, distinguish:

```text
INSTRUCTION
ENFORCEMENT MECHANISM
COMPLIANCE EVIDENCE
```

**Failure mode prevented**

Claiming automation or guarantees that exist only as prose.

**Applies when**

A contract uses words such as:

- enforce;
- prevent;
- automatically;
- guarantee;
- persist;
- isolate;
- independently verify.

**Enforcement mechanism**

The contract must name the concrete mechanism or label the rule advisory.

**Required evidence**

Evidence from that mechanism.

**Response to violation**

Rewrite the claim accurately.

Example:

Bad:

> The harness prevents two tasks from becoming active.

Accurate if no controller exists:

> The instructions require WIP=1, but enforcement is advisory until a state controller rejects a second activation.

---

## HR-22 — Verification evidence has a validity scope

**Strength:** Principle

**Requirement**

Evidence must identify the artifact/version/environment to which it applies.

**Failure mode prevented**

Reusing an old passing result after relevant code or environment changed.

**Applies when**

A task is marked verified or passing.

**Enforcement mechanism**

Evidence record includes, where applicable:

```text
ARTIFACT_VERSION:
[commit/build/version]

ENVIRONMENT:
[...]

CHECK:
[...]

RESULT:
[...]

TIME:
[...]

INVALIDATED_BY:
[change classes]
```

**Required evidence**

The evidence matches the current artifact and relevant environment.

**Response to violation**

Mark verification stale and require re-verification.

---

# 9. Handoffs

## HR-23 — Every session ends with reconstructable state

**Strength:** Principle

**Requirement**

Before a session ends, durable state must answer:

```text
WHAT WAS ATTEMPTED?
WHAT CHANGED?
WHAT PASSED?
WHAT FAILED?
WHAT IS BLOCKED?
WHAT EVIDENCE EXISTS?
WHAT IS THE NEXT PERMITTED ACTION?
```

**Failure mode prevented**

A future session having to reverse-engineer unfinished work.

**Applies when**

Work is paused, blocked, completed, or transferred.

**Enforcement mechanism**

Use the designated handoff artifact.

**Required evidence**

A fresh reviewer can identify the next action without relying on the prior chat.

**Response to violation**

The session is not handoff-ready.

Repair the state record before claiming a clean handoff.

---

## HR-24 — Handoffs must report failure honestly

**Strength:** Principle

**Requirement**

Do not clean up the wording of an unhealthy project state.

Record failures, partial work, and uncertainty explicitly.

**Failure mode prevented**

False “clean finish” reports that hide unresolved risk.

**Applies when**

A task ends without verified completion.

**Enforcement mechanism**

Handoff states must support explicit:

```text
blocked
verification_failed
partially_recovered
awaiting_decision
```

or equivalent status details, even if the main task-state model remains small.

**Required evidence**

Known failures are present in the handoff.

**Response to violation**

Correct the handoff and downgrade any unsupported completion claim.

---

## HR-25 — Preserve unrelated work during recovery

**Strength:** Principle

**Requirement**

Recovery actions must minimize damage to verified or unrelated work.

Do not reset an entire project when a local rollback is sufficient.

**Failure mode prevented**

A repair attempt destroying valid changes outside the failing scope.

**Applies when**

Rollback, restore, or cleanup is required.

**Enforcement mechanism**

Use project-appropriate isolation and version-control mechanisms.

**Required evidence**

Recovery scope is documented and unrelated verified work remains intact.

**Response to violation**

Stop recovery and escalate before destructive action continues.

Detailed recovery routing belongs in `LOOP_RULES.md`.

---

# 10. Clean Handoff Readiness

## HR-26 — “Clean” means resumable, not perfect

**Strength:** Principle

**Requirement**

A clean handoff means the project is left in an honest, reconstructable, bounded state.

It does **not** require every task to be complete.

A blocked task can have a clean handoff.

**Failure mode prevented**

Agents hiding failures because they believe only a passing project can be handed off cleanly.

**Applies when**

Ending any session.

**Enforcement mechanism**

Handoff checklist:

```text
[ ] Current task state recorded
[ ] Verification evidence recorded
[ ] Known failures recorded
[ ] Next action recorded
[ ] Relevant changes preserved
[ ] No misleading completion claim
```

Where relevant, also include:

```text
[ ] Build/start path known
[ ] Temporary artifacts identified or removed
```

**Required evidence**

The durable state passes the fresh-session test.

**Response to violation**

Repair the handoff artifacts before ending the work unit.

---

# 11. Harness Complexity Control

## HR-27 — Every harness mechanism needs a failure mode

**Strength:** Principle

**Requirement**

Do not add rules, files, reviewers, agents, gates, or automation merely because they are considered “best practice.”

Each must address a concrete risk.

**Failure mode prevented**

- ceremonial process;
- excessive maintenance cost;
- irrelevant context;
- orchestration overhead.

**Applies when**

Adding any harness component.

**Enforcement mechanism**

Record:

```text
MECHANISM:
[...]

FAILURE MODE ADDRESSED:
[...]

COST:
[...]

REMOVE OR REEVALUATE WHEN:
[...]
```

**Required evidence**

The mechanism has a clear purpose and observable contribution.

**Response to violation**

Remove it or downgrade it to optional guidance.

---

## HR-28 — Prefer enforceable invariants over repeated prose

**Strength:** Default

**Requirement**

When a recurring rule can be cheaply and safely enforced by tooling, prefer mechanization over repeatedly instructing the agent.

Examples:

```text
architecture boundary → static check
format requirement     → formatter/linter
API compatibility      → contract test
state limit            → state transition guard
```

**Failure mode prevented**

Repeated instruction failures and review burden.

**Applies when**

The rule is objective and machine-checkable.

**Enforcement mechanism**

Project-dependent automated check.

**Required evidence**

The check runs against the relevant change.

**Response to violation**

Either add the missing mechanism or explicitly document why the rule remains advisory.

---

## HR-29 — Do not automate subjective decisions prematurely

**Strength:** Project addition

**Requirement**

Do not force judgment-heavy criteria into fake binary automation merely to make them “executable.”

**Failure mode prevented**

Bad proxies, Goodhart effects, and misleading pass/fail signals.

**Applies when**

The criterion depends materially on usability, architectural judgment, ambiguity, policy interpretation, or qualitative tradeoffs.

**Enforcement mechanism**

Assign an appropriate evaluator:

```text
human reviewer
independent model evaluator
structured rubric
external authority
mixed automated + human gate
```

**Required evidence**

The evaluator's decision and supporting evidence are recorded.

**Response to violation**

Replace the weak proxy with an appropriate evaluation mechanism.

---

# 12. Worked Example

## Scenario

Future request:

> “Add authentication to the application.”

This is not yet builder-ready.

A harness analysis might produce:

```text
OBJECTIVE:
Allow an existing user to authenticate with email and password.

INITIAL SCOPE:
Login only.

OUT OF SCOPE:
Registration
Password reset
OAuth
Email verification
Global error-handling refactor

REQUIRED CONTEXT:
[AUTH_REQUIREMENTS]
[CURRENT_ARCHITECTURE]
[DATABASE_SCHEMA]
[TESTING_GUIDE]

SOURCE AUTHORITY:
Required behavior → [AUTH_REQUIREMENTS]
Current code → repository
Current schema → migration files / live dev database as designated
Runtime behavior → dev environment

READINESS:
Application starts       → unverified
Test suite runs          → unverified
Auth test fixture exists → unknown
Logs observable          → unverified

TOOLS:
Read repository → permitted
Modify application/test files → permitted after readiness
Production deployment → prohibited
Database destructive reset → approval required

WIP:
Default limit = 1 active implementation task

PERSISTENCE:
Task state → [CURRENT_TASK_STATE]
Decisions → [DECISION_LOG]
Verification → [VERIFICATION_RECORD]

HANDOFF REQUIREMENT:
Record exact current state, test evidence, blockers, and next permitted action.
```

Important:

This document does **not** say how many implementation retries are allowed after the login test fails.

That belongs in `LOOP_RULES.md`.

---

# 13. Minimal Harness Design Checklist

Before declaring an execution contract harness-ready, confirm:

```text
CONTEXT
[ ] Required context is identified.
[ ] Irrelevant context is excluded or deferred.

AUTHORITY
[ ] Consequential fact domains have authoritative sources.
[ ] Freshness and conflicts are handled.

READINESS
[ ] Required environment capabilities are known.
[ ] Unverified capabilities are not treated as available.

TOOLS
[ ] Read/write/destructive boundaries are explicit.
[ ] High-impact actions have appropriate gates.

SCOPE
[ ] The active task is bounded.
[ ] Parallelism, if any, is justified and isolated.

STATE
[ ] Durable state exists outside conversation history.
[ ] Specification, status, and evidence are distinct.

HANDOFF
[ ] A fresh session can reconstruct current reality.
[ ] Failures and blockers are recorded honestly.

COMPLEXITY
[ ] Every harness mechanism addresses a concrete failure mode.
[ ] Advisory rules are not described as enforced.
```

A failure in this checklist does not necessarily invalidate the whole project.

It identifies a readiness gap that must be resolved, explicitly accepted, or carried forward as a blocker.
