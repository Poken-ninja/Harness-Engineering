# LOOP_RULES.md

## Purpose

This document defines the reusable control-flow rules for executing bounded tasks.

It governs:

- task states and transitions;
- task entry conditions;
- action → verification → repair cycles;
- failure classification;
- retry and resource budgets;
- stopping and blocking conditions;
- escalation;
- checkpoints, rollback, and recovery;
- verification invalidation.

It does not define project-specific task content, tool permissions, or source authority. Those belong in the execution contract and `HARNESS_RULES.md`.

---

# 1. Rule Strength

## Principle

A control property that should hold across projects.

## Default

The normal choice when the project provides no stronger reason to do otherwise.

## Project-dependent choice

A value that must be selected for the actual project.

Typical placeholders include:

```text
[RETRY_LIMIT]
[TIME_OR_COST_BUDGET]
[VERIFICATION_METHOD]
[RECOVERY_POINT]
[ESCALATION_OWNER]
[FAILURE_CLASSIFICATION_RULES]
```

Numerical examples in this document are defaults or illustrations unless explicitly marked otherwise.

---

# 2. Core Task State Model

## LR-01 — Every executable task has one authoritative state

**Strength:** Principle

Use this minimal lifecycle:

```text
not_started
    |
    v
  active
   /  \
  v    v
blocked passing
```

`not_started` means the task has not entered execution.

`active` means execution is currently permitted.

`blocked` means execution cannot safely or productively continue without an external change, decision, capability, or dependency.

`passing` means all currently valid acceptance criteria have supporting verification evidence.

**Failure mode prevented**

Ambiguous statuses such as “mostly done,” “probably fixed,” or “90% complete.”

**Applicability**

Every bounded implementation task.

**Enforcement mechanism**

The project's task-state store is the authoritative state record.

Where no state controller exists, transitions are advisory and must not be described as automatically enforced.

**Required evidence**

Every state transition identifies its triggering event.

**Response to violation**

Reconstruct the state from evidence. If uncertainty remains, use the less-complete state.

---

# 3. Exceptional Conditions Are Not New Task States

## LR-02 — Keep the state machine small

**Strength:** Default

Do not create new primary states for every execution condition.

Represent conditions separately, for example:

```text
state: blocked
reason: external_dependency
```

or:

```text
state: active
condition: verification_failed
attempt: 2
```

Useful condition labels may include:

```text
verification_failed
awaiting_decision
dependency_unavailable
environment_broken
budget_exhausted
evidence_stale
recovery_in_progress
```

**Failure mode prevented**

A state machine so complex that transitions themselves become hard to reason about.

**Applicability**

Whenever new status labels are proposed.

**Enforcement mechanism**

Primary lifecycle state and diagnostic condition are stored separately.

**Required evidence**

Every condition maps back to one of the four primary states.

**Response to violation**

Collapse unnecessary states into state + reason.

---

# 4. Task Entry Gate

## LR-03 — A task may enter `active` only when its entry conditions are satisfied

**Strength:** Principle

Before activation, confirm:

```text
TASK_ID
OBJECTIVE
IN_SCOPE
OUT_OF_SCOPE
DEPENDENCIES
ACCEPTANCE_CRITERIA
VERIFICATION_METHODS
REQUIRED_CONTEXT
REQUIRED_TOOLS
REQUIRED_PERMISSIONS
READINESS_STATUS
RETRY_BUDGET
RECOVERY_POINT
```

Project-specific values may remain unknown only if they are not required to begin safely.

**Failure mode prevented**

Starting implementation before success, permissions, dependencies, or verification are defined.

**Applicability**

Every transition from `not_started` to `active`.

**Enforcement mechanism**

Entry-gate validation performed against the task contract.

**Required evidence**

All mandatory entry fields are populated and prerequisite readiness checks have passed.

**Response to violation**

Keep the task `not_started`, or mark it `blocked` if an external dependency prevents satisfying the entry gate.

---

# 5. Acceptance Criteria Must Be Independently Judgeable

## LR-04 — Every acceptance criterion needs an evaluator and evidence type

**Strength:** Principle

For each criterion, define:

```text
CRITERION:
[observable requirement]

EVALUATOR:
[automated check | human | independent agent | external system]

EVIDENCE:
[expected result]

INVALIDATED_BY:
[relevant changes]
```

An executable command is preferred when the behavior is objectively machine-checkable, but is not universally required.

**Failure mode prevented**

Equating “implemented” with “verified.”

**Applicability**

Every acceptance criterion.

**Enforcement mechanism**

A task cannot enter `passing` until every criterion has valid evidence.

**Required evidence**

Actual output from the designated evaluator.

**Response to violation**

Keep the task `active` or `blocked`; do not infer success.

---

# 6. The Basic Execution Cycle

## LR-05 — Execution follows action → verification → classification

**Strength:** Principle

The core loop is:

```text
ENTRY
  |
  v
ACTION
  |
  v
VERIFY
 / | \
/  |  \
pass fixable non-fixable/unknown
 |     |        |
 v     v        v
DONE  REPAIR   BLOCK/ESCALATE
        |
        v
      VERIFY
```

An action attempt must be followed by verification before another significant modification is made, unless the task contract explicitly defines a batch operation.

**Failure mode prevented**

Large accumulations of unverified changes and blind “keep coding” behavior.

**Applicability**

Every active task.

**Enforcement mechanism**

Attempt records distinguish action from verification.

**Required evidence**

Each repair attempt is linked to the failure evidence that motivated it.

**Response to violation**

Stop adding changes, run the required verification, and re-establish the current failure state.

---

# 7. Verification Is a Gate, Not Commentary

## LR-06 — The producer does not control the pass transition

**Strength:** Principle

The agent or process performing implementation may propose that work is complete, but the `active → passing` transition is determined by the designated evaluator.

**Failure mode prevented**

Premature completion and self-certification.

**Applicability**

Every completion transition.

**Enforcement mechanism**

Project-dependent verifier, test runner, human reviewer, or separate evaluator.

**Required evidence**

Passing outputs for all current acceptance criteria.

**Response to violation**

Revert the unsupported transition to `active`.

---

# 8. Independence Must Be Real When Required

## LR-07 — Self-review is not independent review

**Strength:** Principle

A second review pass in the same context is self-review.

When independence materially reduces risk, use one or more of:

```text
fresh evaluator context
different agent/session
deterministic test
external system
human reviewer
separate verification environment
```

**Failure mode prevented**

A producer reproducing its own mistaken assumptions during review.

**Applicability**

High-impact changes, ambiguous correctness criteria, security-sensitive work, or tasks whose contract explicitly requires independent evaluation.

**Enforcement mechanism**

Evaluator isolation or external checking mechanism.

**Required evidence**

The verifier's inputs and identity/context demonstrate the intended separation.

**Response to violation**

Relabel the result as self-review and obtain the required independent check before passing.

---

# 9. Failure Classification Before Repair

## LR-08 — Classify a failure before selecting a repair route

**Strength:** Principle

Use the following reusable failure classes:

| Failure class | Meaning | Normal route |
|---|---|---|
| `implementation_defect` | Intended design is sound; implementation is wrong | repair implementation |
| `verification_defect` | Check is broken, flaky, misleading, or inconsistent with criterion | repair/evaluate verification mechanism |
| `context_gap` | Required information is missing | obtain context, then resume |
| `requirement_ambiguity` | Correct behavior cannot be determined | decision/escalation |
| `dependency_failure` | External prerequisite is unavailable or incorrect | block or recover dependency |
| `environment_failure` | Build/test/runtime environment is not trustworthy | repair readiness before feature work |
| `permission_failure` | Required action is not authorized | escalate or redesign |
| `scope_mismatch` | Required repair lies outside the task boundary | create dependency/change scope explicitly |
| `architecture_conflict` | Local repair conflicts with higher-level constraints | escalate/design decision |
| `resource_exhaustion` | Retry, time, cost, or context budget is exhausted | stop and escalate |
| `unknown_failure` | Cause cannot yet be established | diagnosis, not repeated repair |

**Failure mode prevented**

Repeating the same action against the wrong problem.

**Applicability**

Every failed verification or blocked execution step.

**Enforcement mechanism**

Failure record required before a retry beyond the first obvious correction.

**Required evidence**

Observed symptoms and reasoning supporting the classification.

**Response to violation**

Pause retries and diagnose.

**Project addition:** The explicit failure taxonomy is added because the supplied material argues for bounded repair but does not provide a reusable classification system.

---

# 10. Repair Must Target the Diagnosed Cause

## LR-09 — No identical retry without new information

**Strength:** Principle

A failed attempt may be retried only when at least one relevant condition changes, such as:

```text
implementation changed
context improved
dependency recovered
verification corrected
environment repaired
assumption revised
scope decision obtained
```

**Failure mode prevented**

“Repeat until it works” loops.

**Applicability**

After any failed attempt.

**Enforcement mechanism**

Each retry record must state:

```text
PREVIOUS_FAILURE:
[...]

CHANGE_BEFORE_RETRY:
[...]

WHY THIS MAY ALTER THE RESULT:
[...]
```

**Required evidence**

The new attempt differs materially from the failed attempt.

**Response to violation**

Stop the retry sequence and enter diagnosis/escalation.

---

# 11. Retry Budgets

## LR-10 — Every repair loop has a finite budget

**Strength:** Principle

Before execution, specify the applicable budget.

The budget may be based on:

```text
attempt count
elapsed time
tool/runtime cost
token/inference cost
risk exposure
combination of the above
```

Example default for a small, well-understood repair:

```text
RETRY_LIMIT: 3
```

This is an illustration, not a universal requirement.

**Failure mode prevented**

Endless looping and uncontrolled resource consumption.

**Applicability**

Any action that may repeat following failure.

**Enforcement mechanism**

Attempt counter and resource accounting.

**Required evidence**

Current attempt count and remaining budget are recorded.

**Response to violation**

Stop the loop immediately and route to escalation or blocked state.

---

# 12. Retry Budgets Should Shrink With Risk

## LR-11 — High-risk retries require tighter control

**Strength:** Principle

The acceptable retry policy should become more conservative as actions become more destructive, expensive, externally visible, or difficult to reverse.

For example:

```text
local deterministic test fix
→ several bounded retries may be reasonable

production migration
→ retry may require explicit diagnosis and approval before every reattempt
```

**Failure mode prevented**

Repeatedly applying risky operations merely because an automatic retry mechanism exists.

**Applicability**

Tasks with external side effects or limited reversibility.

**Enforcement mechanism**

Risk-specific retry rules in the execution contract.

**Required evidence**

Retry approval or preconditions appropriate to the operation's risk.

**Response to violation**

Stop and escalate before another high-impact attempt.

---

# 13. Verification Failures Consume the Budget Only When Work Was Attempted

## LR-12 — Distinguish attempts from diagnostic checks

**Strength:** Default

A diagnostic observation that does not materially modify the candidate solution should not necessarily consume an implementation retry.

Example:

```text
test fails
inspect logs
inspect database state
identify root cause
```

This may belong to one repair attempt.

**Failure mode prevented**

Artificially exhausting budgets through harmless investigation.

**Applicability**

Tasks with multi-step diagnosis.

**Enforcement mechanism**

Record separate counters where useful:

```text
IMPLEMENTATION_ATTEMPTS
DIAGNOSTIC_ACTIONS
```

**Required evidence**

The execution trace clearly distinguishes modifications from observations.

**Response to violation**

Correct accounting rather than silently increasing the budget.

---

# 14. Escalation Conditions

## LR-13 — Escalate on structural uncertainty, not only exhaustion

**Strength:** Principle

Escalation is required when any of the following materially affects correctness:

```text
acceptance criteria conflict
authoritative sources conflict
required permission is unavailable
high-impact action requires approval
architecture decision is unresolved
repair would exceed scope
failure class remains unknown after diagnosis
the same failure recurs despite materially different repairs
required verifier is unavailable
risk exceeds the task's authorized boundary
```

**Failure mode prevented**

An agent making unauthorized product, architectural, policy, or risk decisions merely to keep moving.

**Applicability**

Whenever the builder lacks authority or sufficient information to select a safe repair.

**Enforcement mechanism**

Explicit escalation route in the task contract.

**Required evidence**

The blocker and requested decision are recorded precisely.

**Response to violation**

Move the task to `blocked`.

---

# 15. Escalation Must Be Actionable

## LR-14 — A blocked handoff identifies the decision needed

**Strength:** Principle

Do not escalate with:

> “Need help.”

Record:

```text
BLOCKER:
[observable issue]

WHY EXECUTION CANNOT CONTINUE:
[...]

DECISION OR INPUT REQUIRED:
[...]

OPTIONS ALREADY ESTABLISHED:
[...]

CONSEQUENCE OF EACH OPTION:
[...]

NEXT ACTION AFTER RESOLUTION:
[...]
```

**Failure mode prevented**

Humans having to rediscover the entire task before resolving a blocker.

**Applicability**

Every blocked task requiring external input.

**Enforcement mechanism**

Blocked-state schema.

**Required evidence**

A reviewer can identify exactly what would unblock execution.

**Response to violation**

Improve the blocker record before handoff.

---

# 16. Stop Conditions

## LR-15 — Every loop must define success, blocked, and abort exits

**Strength:** Principle

Every task loop needs three categories of termination.

### Success

All current acceptance criteria have valid evidence.

Route:

```text
active → passing
```

### Blocked

The task cannot proceed without an external change or decision.

Route:

```text
active → blocked
```

### Abort / controlled stop

Continuing would be unsafe, wasteful, or outside authorization.

Examples:

```text
retry budget exhausted
environment integrity uncertain
recovery impossible within permitted scope
unexpected destructive risk
verification mechanism cannot be trusted
```

The primary task state may become `blocked` with an appropriate reason.

**Failure mode prevented**

Loops that have only “keep trying” and “success” paths.

**Applicability**

Every repeatable task.

**Enforcement mechanism**

Stop conditions written before execution.

**Required evidence**

Termination record identifies which condition fired.

**Response to violation**

Treat the loop specification as incomplete.

---

# 17. Passing Means Current Evidence, Not Permanent Truth

## LR-16 — Verification may be invalidated

**Strength:** Principle

A task marked `passing` remains historically verified for the version tested, but its evidence may become stale.

Examples of invalidation:

```text
relevant code changed
dependency changed
shared interface changed
environment changed materially
acceptance criterion changed
test itself changed
authoritative requirement changed
integration altered behavior
```

**Failure mode prevented**

Treating old evidence as proof of current correctness.

**Applicability**

Any previously verified behavior affected by later changes.

**Enforcement mechanism**

Dependency-aware invalidation rules.

**Required evidence**

A change-impact check determines whether prior evidence remains valid.

**Response to violation**

Mark the verification record stale and require re-verification before relying on it.

**Correction to supplied material:** “Passing is irreversible” is not adopted as a universal rule. Historical evidence is immutable; current verification validity is not.

---

# 18. Verification Scope Should Match Risk

## LR-17 — Use the narrowest adequate check, then the required broader gate

**Strength:** Default

During repair, use fast local checks when they provide useful feedback.

Before passing, use the verification level required by the acceptance criterion and integration risk.

Example:

```text
edit parser
→ parser unit test
→ integration test
→ required end-to-end flow
```

**Failure mode prevented**

Either extreme:

- running expensive full-system tests after every keystroke; or
- declaring success from a narrow unit test when system behavior matters.

**Applicability**

Tasks with multiple verification layers.

**Enforcement mechanism**

Task contract identifies:

```text
FAST_FEEDBACK_CHECKS
PASSING_GATE
```

**Required evidence**

The final passing gate has been run after the last relevant modification.

**Response to violation**

Run the missing broader verification.

---

# 19. Repair Feedback Should Be Actionable

## LR-18 — Verification should expose useful failure evidence

**Strength:** Default

Where controllable, failure output should provide enough information to distinguish likely repair routes.

Useful evidence includes:

```text
expected behavior
observed behavior
failing component or boundary
relevant logs
reproduction conditions
artifact/version tested
```

Do not force speculative repair instructions into the error message when the cause is not known.

**Failure mode prevented**

Blind retries based on vague “failed” signals.

**Applicability**

Tests, CI checks, evaluators, and runtime monitors designed for agent consumption.

**Enforcement mechanism**

Verification-output design.

**Required evidence**

A failed check exposes enough evidence for diagnosis.

**Response to violation**

Improve observability or route to diagnostic work.

---

# 20. Checkpoints

## LR-19 — Checkpoint at meaningful recovery boundaries

**Strength:** Principle

Persist state when losing work would materially increase recovery cost.

Typical checkpoint moments:

```text
before a high-risk change
after a coherent implementation milestone
after successful verification
before human approval
before integration
before session handoff
```

Do not checkpoint every trivial action unless the platform requires it.

**Failure mode prevented**

Restarting large amounts of valid work after localized failure.

**Applicability**

Long-running or stateful tasks.

**Enforcement mechanism**

Project-dependent version control, persisted state, snapshots, or workflow checkpoints.

**Required evidence**

The recovery point exists and can be identified.

**Response to violation**

Create a checkpoint before proceeding into higher-risk work where possible.

---

# 21. Recovery

## LR-20 — Recover to the smallest trustworthy boundary

**Strength:** Principle

When failure corrupts the current path, recover only as far as necessary.

Preferred order:

```text
repair current step
↓
restore current task checkpoint
↓
restore task-start checkpoint
↓
restore broader project checkpoint
```

Escalate before broader destructive rollback when unrelated verified work may be affected.

**Failure mode prevented**

Destroying good work to repair a local failure.

**Applicability**

Any rollback or restoration.

**Enforcement mechanism**

Checkpoint hierarchy and version-control boundaries.

**Required evidence**

The selected recovery point predates the failure and preserves unrelated valid work.

**Response to violation**

Stop recovery and reassess the rollback scope.

---

# 22. Recovery Must Re-Establish Trust

## LR-21 — After recovery, rerun invalidated checks

**Strength:** Principle

Restoring files or state is not itself proof that the system is healthy.

After recovery:

```text
restore
→ confirm environment/state integrity
→ rerun affected verification
→ resume
```

**Failure mode prevented**

Continuing from a rollback point whose actual consistency was never checked.

**Applicability**

Every non-trivial rollback.

**Enforcement mechanism**

Recovery verification defined by the task contract.

**Required evidence**

Affected readiness and behavioral checks pass.

**Response to violation**

Remain blocked or continue recovery.

---

# 23. Recovery Cannot Hide Failed Attempts

## LR-22 — Preserve failure history

**Strength:** Principle

Do not erase evidence that a failed path occurred merely because the code was rolled back.

Record:

```text
ATTEMPT
FAILURE
CLASSIFICATION
RECOVERY ACTION
RESULT
```

**Failure mode prevented**

Future sessions repeating previously failed approaches.

**Applicability**

Any failed attempt that produced useful diagnostic knowledge.

**Enforcement mechanism**

Attempt log or progress record.

**Required evidence**

The failure remains discoverable after rollback.

**Response to violation**

Reconstruct the failure history before further retries if material.

---

# 24. Scope Changes During Execution

## LR-23 — Scope changes require a contract change

**Strength:** Principle

A repair that materially changes:

```text
objective
acceptance criteria
dependencies
affected subsystem
permissions
risk
verification method
```

is not merely another retry.

It is a task-contract change.

**Failure mode prevented**

Using “repair” as a hidden path to expand implementation scope.

**Applicability**

Whenever the diagnosed solution exceeds the original boundary.

**Enforcement mechanism**

Pause the loop and update the contract before implementation continues.

**Required evidence**

Changed scope and rationale are recorded.

**Response to violation**

Stop the out-of-scope repair and restore/segregate unauthorized changes where practical.

---

# 25. Dependency Discovery

## LR-24 — Newly discovered dependencies are classified before activation

**Strength:** Principle

When a task discovers prerequisite work, determine whether the dependency is:

```text
already satisfied
resolvable inside current scope
a new prerequisite task
external blocker
optional/non-blocking
```

Do not silently start the new work.

**Failure mode prevented**

Nested scope expansion and multiple active tasks.

**Applicability**

Whenever implementation reveals unplanned prerequisite work.

**Enforcement mechanism**

Dependency record plus WIP rule from `HARNESS_RULES.md`.

**Required evidence**

The dependency has a route and state.

**Response to violation**

Pause the current branch of work until the dependency is formally routed.

---

# 26. Parallel Loops

## LR-25 — Parallel loops require independent ownership and an integration gate

**Strength:** Principle

Parallel execution is allowed only when the harness has already justified it.

Each parallel branch must define:

```text
owned scope
owned state
verification
budget
recovery boundary
integration point
```

Passing individual branches does not imply the integrated result passes.

**Failure mode prevented**

Several locally successful agents producing a globally inconsistent system.

**Applicability**

Multi-agent or graph execution.

**Enforcement mechanism**

Isolation plus post-integration verification.

**Required evidence**

Integrated acceptance criteria pass after branch combination.

**Response to violation**

Do not mark the parent objective passing.

---

# 27. Parent and Child Loop Completion

## LR-26 — Child success does not automatically imply parent success

**Strength:** Principle

In graphs or decomposed tasks:

```text
child A passing
child B passing
child C passing
```

does not prove the parent behavior unless the parent integration criteria are also verified.

**Failure mode prevented**

Mistaking component correctness for system correctness.

**Applicability**

Tasks decomposed into subtasks or graph nodes.

**Enforcement mechanism**

Parent-level acceptance criteria and integration verification.

**Required evidence**

Parent evaluator passes against the integrated result.

**Response to violation**

Keep the parent objective active or blocked.

---

# 28. Human Approval Is a Pause, Not a Pass

## LR-27 — Approval gates do not replace technical verification

**Strength:** Principle

When a human approval node is required, distinguish:

```text
TECHNICAL VERIFICATION
HUMAN APPROVAL
FINAL TRANSITION
```

A human may approve a tradeoff without proving the system works.

A test may prove behavior without authorizing deployment.

**Failure mode prevented**

Conflating permission, judgment, and correctness.

**Applicability**

Tasks with human-in-the-loop controls.

**Enforcement mechanism**

Separate gate records.

**Required evidence**

All required gate types are satisfied.

**Response to violation**

Do not proceed to the gated action.

---

# 29. Unknown Failures Trigger Diagnosis

## LR-28 — Diagnosis has its own bounded loop

**Strength:** Project addition

When failure cause is unknown, temporarily switch from repair mode to diagnosis mode.

Diagnosis loop:

```text
OBSERVE
  ↓
FORM HYPOTHESIS
  ↓
RUN DISCRIMINATING CHECK
  ↓
UPDATE FAILURE CLASS
```

Set a diagnosis budget.

The goal is not “fix something”; it is “reduce uncertainty enough to select the correct repair route.”

**Failure mode prevented**

Random code edits against unexplained symptoms.

**Applicability**

`unknown_failure` or repeated failed repairs.

**Enforcement mechanism**

Diagnostic actions are recorded separately from implementation attempts.

**Required evidence**

The diagnosis either:

- identifies a plausible failure class; or
- exhausts its budget and escalates.

**Response to violation**

Stop speculative repair.

---

# 30. Stop Escalation From Becoming Another Endless Loop

## LR-29 — Escalation itself has an owner and next condition

**Strength:** Principle

A task cannot remain indefinitely “waiting for someone.”

Record:

```text
ESCALATION_OWNER:
[...]

INPUT_REQUIRED:
[...]

RESUME_CONDITION:
[...]

FALLBACK_IF_UNRESOLVED:
[defer | cancel | redesign | remain blocked]
```

**Failure mode prevented**

Zombie tasks with no defined route back to execution.

**Applicability**

Every escalated blocker.

**Enforcement mechanism**

Persistent blocker record.

**Required evidence**

A specific event can be identified that would permit resumption.

**Response to violation**

Treat the task as unowned blocked work until corrected.

---

# 31. Task Resumption

## LR-30 — Resume from durable evidence, not remembered conversation

**Strength:** Principle

Before resuming a paused task, reconstruct:

```text
current contract version
current state
last verified artifact
known failures
remaining retry budget
active blocker status
next permitted action
```

**Failure mode prevented**

Repeating old attempts or continuing from obsolete assumptions.

**Applicability**

Any resumed session or reassigned task.

**Enforcement mechanism**

Persistent state and handoff record.

**Required evidence**

The new session can state why its next action is currently permitted.

**Response to violation**

Perform reconstruction before changing the project.

---

# 32. Verification Invalidation After Repair

## LR-31 — Any repair may invalidate neighboring evidence

**Strength:** Principle

After a repair, determine its impact radius.

Classify prior checks as:

```text
still_valid
must_rerun
unknown_validity
```

**Failure mode prevented**

Fixing one behavior while unknowingly breaking another and retaining stale “passing” status.

**Applicability**

Changes to shared components, interfaces, dependencies, or system-wide configuration.

**Enforcement mechanism**

Change-impact analysis defined at the level appropriate to the project.

**Required evidence**

Affected checks are rerun before dependent tasks rely on them.

**Response to violation**

Downgrade affected verification to stale.

---

# 33. Completion Record

## LR-32 — A passing task leaves a reproducible evidence record

**Strength:** Principle

The completion record should contain enough information to understand what passed.

Example:

```text
TASK_ID:
F12

STATE:
passing

ARTIFACT:
[commit/version]

ACCEPTANCE_CRITERIA:
AC1 → PASS → [evidence]
AC2 → PASS → [evidence]
AC3 → PASS → [evidence]

VERIFICATION_ENVIRONMENT:
[...]

ATTEMPTS_USED:
2 / 3

KNOWN_LIMITATIONS:
[...]

INVALIDATION_TRIGGERS:
[...]
```

**Failure mode prevented**

“Done” statuses that cannot be audited later.

**Applicability**

Every successful task.

**Enforcement mechanism**

Completion evidence stored with durable state.

**Required evidence**

References resolve to actual verification results.

**Response to violation**

The task is not audit-ready; repair the record.

---

# 34. Worked Example

## Task

```text
TASK_ID:
F12

OBJECTIVE:
An existing user can log in with valid credentials.

ACCEPTANCE:
AC1 Valid credentials return authenticated session.
AC2 Invalid credentials do not authenticate.
AC3 Existing account flow continues to work end-to-end.

RETRY_LIMIT:
3 implementation attempts.

RECOVERY_POINT:
Task-start git checkpoint.

ESCALATION_OWNER:
[PROJECT_OWNER]
```

Initial state:

```text
state: not_started
```

Entry gate passes:

```text
environment ready
required files available
verification runnable
permissions confirmed
```

Transition:

```text
not_started → active
```

### Attempt 1

Action:

```text
implement login handler
```

Verification:

```text
AC1 FAIL
Observed: HTTP 500
Expected: authenticated session
```

Classification:

```text
implementation_defect
```

Diagnosis:

```text
password comparison receives null hash
```

Retry change:

```text
correct user-record lookup and null handling
```

### Attempt 2

Verification:

```text
AC1 PASS
AC2 PASS
AC3 FAIL
```

Failure:

```text
existing session refresh now returns 401
```

Classification:

```text
implementation_defect
```

Repair:

```text
restore refresh-token compatibility without expanding scope
```

### Attempt 3

Verification:

```text
AC1 PASS
AC2 PASS
AC3 PASS
```

Independent required E2E gate:

```text
PASS
```

Transition:

```text
active → passing
```

The task is passing because evidence exists, not because three attempts were used.

If attempt 3 had failed, the correct route would have been:

```text
budget exhausted
→ stop
→ record failure
→ classify remaining blocker
→ escalate or redesign
```

It would **not** automatically receive attempt 4.

---

# 35. Compact Loop Contract

A future task's execution loop should be expressible in this form:

```text
TRIGGER:
[what activates this loop]

ENTRY CONDITIONS:
[what must already be true]

INPUTS:
[required state/context]

PERMITTED ACTION:
[bounded work]

OUTPUT:
[artifact/state produced]

VERIFICATION:
[evaluator + evidence]

PASS ROUTE:
[next state/node]

FAILURE CLASSES:
[classification rules]

REPAIR ROUTE:
[what may be retried]

RETRY BUDGET:
[count/time/cost/risk]

BLOCK CONDITIONS:
[external dependencies]

ESCALATION:
[owner + input needed]

STOP CONDITIONS:
[success / blocked / abort]

CHECKPOINT:
[recovery boundary]

RECOVERY:
[how to restore safely]

INVALIDATION:
[changes that require re-verification]
```

If a meaningful field cannot yet be determined, mark it as an unresolved project decision rather than inventing it.

---

# 36. Loop Quality Checklist

Before accepting a loop design, verify:

| Check | Requirement |
|---|---|
| Entry | The task cannot start before prerequisites are satisfied. |
| Scope | The permitted action is bounded. |
| Verification | Success is determined by observable evidence. |
| Evaluator | The evaluator is identified and suitably independent where needed. |
| Failure | Failures are classified before repeated repair. |
| Retry | Repetition has a finite budget. |
| Learning | Identical failed attempts are not repeated without changed information. |
| Stop | Success, blocked, and abort exits exist. |
| Escalation | Decisions outside the builder's authority have a defined route. |
| Checkpoint | Meaningful recovery points exist. |
| Recovery | Rollback preserves unrelated valid work. |
| Persistence | A fresh session can resume from durable state. |
| Invalidation | Later changes can trigger re-verification. |
| Complexity | The loop is no more elaborate than the task requires. |

A loop is not ready merely because arrows can be drawn between actions.

It is ready when a fresh executor can determine:

**whether it may start, what it may do, how success is judged, what happens after failure, how many times repair may be attempted, when to stop, and how to recover without inventing missing control logic.**
