# EXECUTION_CONTRACT_TEMPLATE.md

## Purpose

Use this template whenever a new idea, rough prompt, existing plan, or requested workflow must be transformed into a builder-ready execution contract.

The completed contract specifies:

- what outcome is required;
- what is known versus assumed;
- what sources control which facts;
- what must be operational before implementation;
- what work units exist;
- what one task may do at a time;
- how success is verified;
- what happens when execution fails;
- how work is resumed safely;
- what remains unresolved.

A completed contract is a **specification**, not evidence that implementation occurred.

Do not claim execution, automation, persistence, independent review, or verification unless corresponding evidence exists.

---

# 0. Contract Metadata

```text
CONTRACT_ID:
[unique identifier]

PROJECT:
[project name or placeholder]

CONTRACT_VERSION:
[version]

STATUS:
[draft | ready_for_builder | blocked]

CREATED_FROM:
[user idea / existing prompt / specification / plan / other]

AUTHORITATIVE_PROJECT_INSTRUCTIONS:
[location or reference]

LAST_REVIEWED:
[date/version if applicable]
```

## Contract Status Meaning

`draft`  
The specification is incomplete or still contains unresolved design choices.

`ready_for_builder`  
The contract satisfies `QUALITY_RUBRIC.md` and has no unresolved gap that prevents the first bounded task from starting.

`blocked`  
The contract cannot safely become executable until a named decision, capability, or source is supplied.

Do not use `ready_for_builder` to imply the product itself is implemented.

---

# 1. Objective

## 1.1 Problem

```text
PROBLEM:
[What problem is being solved?]
```

## 1.2 Intended User or Beneficiary

```text
USERS:
[Who experiences the problem or consumes the result?]
```

## 1.3 Desired Outcome

Describe observable behavior, not implementation preference.

```text
OUTCOME:
[What should become true when the work succeeds?]
```

## 1.4 Success Criteria

```text
SUCCESS_CRITERIA:
- SC1: [observable result]
- SC2: [observable result]
- SC3: [observable result]
```

Each criterion must later map to evidence in Section 9.

## 1.5 Scope

```text
IN_SCOPE:
- [...]

OUT_OF_SCOPE:
- [...]
```

Explicit exclusions should include likely adjacent work when scope creep is predictable.

## 1.6 Constraints

```text
CONSTRAINTS:
- [technical]
- [security]
- [business]
- [time/resource]
- [compatibility]
- [policy]
```

---

# 2. Facts, Assumptions, Decisions, and Unknowns

Do not mix these categories.

## 2.1 Facts

| ID | Fact | Authoritative source | Freshness requirement |
|---|---|---|---|
| F1 | [...] | [...] | [...] |

Only place supported statements here.

---

## 2.2 Assumptions

| ID | Assumption | Why needed | Reversible? | Invalidated by |
|---|---|---|---|---|
| A1 | [...] | [...] | yes/no | [...] |

Prefer assumptions that can be cheaply reversed.

An assumption must not be presented as verified fact elsewhere in the contract.

---

## 2.3 Decisions

| ID | Decision | Alternatives considered | Rationale | Owner | Revisit when |
|---|---|---|---|---|---|
| D1 | [...] | [...] | [...] | [...] | [...] |

Use this section for actual choices, not observations.

---

## 2.4 Unknowns

| ID | Unknown | Why it matters | Blocks execution? | Resolution route |
|---|---|---|---|---|
| U1 | [...] | [...] | yes/no | [...] |

If an unknown materially changes safety, scope, architecture, or acceptance criteria, it normally blocks the affected task.

---

# 3. Source-of-Truth and Context Map

Follow `KNOWLEDGE_MAP.md` and `HARNESS_RULES.md`.

## 3.1 Domain Authority

| Domain | Authoritative source | Secondary source | Conflict rule |
|---|---|---|---|
| Requirements | [...] | [...] | [...] |
| Current code | [...] | [...] | [...] |
| Architecture | [...] | [...] | [...] |
| Runtime behavior | [...] | [...] | [...] |
| Verification status | [...] | [...] | [...] |
| External/current facts | [...] | [...] | [...] |

Add or remove rows according to the project.

Do not declare the repository universally authoritative unless that is truly correct for every listed domain.

---

## 3.2 Required Context for the First Active Task

```text
REQUIRED_CONTEXT:
- [source]: [why required]
- [source]: [why required]
```

## 3.3 Context Available on Demand

```text
ON_DEMAND_CONTEXT:
- [source]
  Consult when: [trigger]
```

## 3.4 Explicitly Excluded Context

Use only where exclusion prevents confusion or stale guidance.

```text
EXCLUDED_CONTEXT:
- [source]
  Reason: [...]
```

---

# 4. Operational Readiness

This section determines whether implementation may begin.

## 4.1 Required Capabilities

| Capability | Required? | Status | Evidence | Owner/Resolution |
|---|---:|---|---|---|
| Repository access | yes/no | available/unverified/unavailable | [...] | [...] |
| Start relevant system | yes/no | [...] | [...] | [...] |
| Run verification | yes/no | [...] | [...] | [...] |
| Observe runtime/logs | yes/no | [...] | [...] | [...] |
| Persist task state | yes/no | [...] | [...] | [...] |
| Checkpoint/rollback | yes/no | [...] | [...] | [...] |
| External service access | yes/no | [...] | [...] | [...] |
| Independent evaluator | yes/no | [...] | [...] | [...] |

Do not mark a capability available merely because the contract requires it.

---

## 4.2 Initialization Work

If readiness is incomplete:

```text
INITIALIZATION_REQUIRED:
yes | no
```

If yes:

```text
INITIALIZATION_OBJECTIVE:
[What operating capability must be established?]

INITIALIZATION_EXCLUSIONS:
- No product feature implementation unless required solely to prove readiness.

INITIALIZATION_ACCEPTANCE:
- [system starts]
- [required check runs]
- [relevant failure output is observable]
- [state can be reconstructed by a fresh session]

INITIALIZATION_EVIDENCE:
[required evidence]
```

Initialization is complete only when the required capabilities are demonstrated.

---

# 5. Tools, Permissions, and Boundaries

## 5.1 Permitted Capabilities

```text
PERMITTED:
- [read operation]
- [write operation]
- [tool or environment]
```

## 5.2 Prohibited Capabilities

```text
PROHIBITED:
- [...]
```

## 5.3 Approval-Gated Capabilities

| Action | Approval/gate | Evidence required |
|---|---|---|
| [...] | [...] | [...] |

Separate:

- read;
- write;
- destructive or irreversible;
- externally visible actions.

---

## 5.4 Enforcement Status

For every consequential boundary, identify whether it is actually enforced.

| Boundary | Written requirement | Enforcement mechanism | Status |
|---|---|---|---|
| [...] | [...] | [...] | enforced/advisory/unverified |

Do not use words such as “prevented” or “guaranteed” when the mechanism is only advisory.

---

# 6. Work Decomposition

## 6.1 Task Graph

List meaningful behavioral work units.

| Task ID | Behavior | Dependencies | State |
|---|---|---|---|
| T01 | [...] | [...] | not_started |
| T02 | [...] | T01 | not_started |
| T03 | [...] | [...] | not_started |

Primary states:

```text
not_started
active
blocked
passing
```

Exceptional conditions belong in separate reason fields.

---

## 6.2 WIP Policy

```text
ACTIVE_IMPLEMENTATION_LIMIT:
[default: 1]
```

If greater than one:

```text
PARALLELISM_JUSTIFICATION:
[...]

ISOLATION_MECHANISM:
[...]

STATE_OWNERSHIP:
[...]

INTEGRATION_POINT:
[...]

POST_INTEGRATION_VERIFICATION:
[...]
```

If these cannot be defined, use serial execution.

---

# 7. Bounded Task Contracts

Create one subsection per task.

Only the next executable task needs full operational detail immediately; later tasks may remain summarized until their dependencies stabilize.

---

## Task `[TASK_ID]` — `[TASK_NAME]`

### Objective

```text
[One observable behavior.]
```

### State

```text
not_started | active | blocked | passing
```

### Dependencies

```text
- [...]
```

### In Scope

```text
- [...]
```

### Out of Scope

```text
- [...]
```

### Required Inputs

```text
- [...]
```

### Permitted Changes

```text
- [files/subsystems/resources]
```

### Prohibited Changes

```text
- [...]
```

### Acceptance Criteria

| ID | Criterion | Evaluator | Required evidence |
|---|---|---|---|
| AC1 | [...] | [...] | [...] |
| AC2 | [...] | [...] | [...] |

### Entry Conditions

```text
- [...]
```

### Verification Method

```text
FAST_FEEDBACK:
[...]

PASSING_GATE:
[...]
```

### Evidence Invalidation

```text
INVALIDATED_BY:
- [...]
```

### Recovery Boundary

```text
CHECKPOINT:
[...]

ROLLBACK_SCOPE:
[...]
```

---

# 8. Execution Loop Contract

Fill this section for every task that can retry, branch, wait, or recover.

## Loop `[LOOP_ID]`

### Trigger

```text
[What starts the loop?]
```

### Required Inputs

```text
- [...]
```

### Entry Conditions

```text
- [...]
```

### Permitted Action

```text
[Exactly what this loop may attempt.]
```

### Expected Output

```text
[Artifact or state produced.]
```

### Verification

```text
EVALUATOR:
[...]

CHECK:
[...]

PASS_EVIDENCE:
[...]
```

### Pass Route

```text
[task/node/state]
```

### Failure Classification

Allowed classes should normally come from `LOOP_RULES.md`.

```text
implementation_defect → [...]
verification_defect   → [...]
context_gap           → [...]
requirement_ambiguity → [...]
dependency_failure    → [...]
environment_failure   → [...]
permission_failure    → [...]
scope_mismatch        → [...]
architecture_conflict → [...]
resource_exhaustion   → [...]
unknown_failure       → [...]
```

Remove classes that cannot apply.

### Repair Route

```text
FAILURE CLASS:
[...]

REPAIR ACTION:
[...]

WHAT MUST CHANGE BEFORE RETRY:
[...]
```

### Retry Budget

```text
ATTEMPT_LIMIT:
[...]

TIME/COST LIMIT:
[...]

HIGH-RISK RETRY RULE:
[...]
```

Do not use unlimited retries.

### Stop Conditions

```text
SUCCESS:
[...]

BLOCKED:
[...]

CONTROLLED_ABORT:
[...]
```

### Escalation

```text
OWNER:
[...]

TRIGGER:
[...]

INPUT OR DECISION REQUIRED:
[...]

RESUME CONDITION:
[...]

FALLBACK IF UNRESOLVED:
[...]
```

### Checkpoint and Recovery

```text
CHECKPOINT:
[...]

RECOVERY ROUTE:
[...]

POST-RECOVERY VERIFICATION:
[...]
```

---

# 9. Acceptance-to-Evidence Matrix

Every top-level success criterion must map to observable evidence.

| Success criterion | Implementing task(s) | Verification | Evaluator | Evidence required | Invalidation condition |
|---|---|---|---|---|---|
| SC1 | [...] | [...] | [...] | [...] | [...] |
| SC2 | [...] | [...] | [...] | [...] | [...] |

A criterion with no evidence path is not yet an executable success criterion.

---

# 10. Independent Evaluation Plan

Use this section only when independence materially matters.

```text
INDEPENDENCE_REQUIRED:
yes | no
```

If yes:

```text
REASON:
[security / high impact / ambiguity / producer bias / policy / other]

PRODUCER:
[...]

EVALUATOR:
[...]

CONTEXT_ISOLATION:
[what evaluator can and cannot see]

SHARED_INPUTS:
[artifacts/state visible to both]

EVALUATOR_AUTHORITY:
[what decision evaluator controls]

DISAGREEMENT_ROUTE:
[...]
```

A second pass in the same context must not be labeled independent evaluation.

---

# 11. Runtime and Process Observability

Specify only the signals necessary to diagnose execution and judge acceptance.

## Runtime Signals

```text
- [logs]
- [health state]
- [trace]
- [test output]
- [external system result]
```

## Process Artifacts

```text
- [task contract]
- [attempt record]
- [decision record]
- [verification result]
```

## Missing Observability

```text
OBSERVABILITY_GAPS:
- [...]

IMPACT:
- [...]

REMEDIATION:
- [...]
```

Do not add telemetry without a concrete diagnostic or verification purpose.

---

# 12. Persistent State Contract

## 12.1 Durable Records

| Record | Purpose | Location | Writer | Reader |
|---|---|---|---|---|
| Current task state | Resume execution | [...] | [...] | [...] |
| Decision log | Preserve decisions | [...] | [...] | [...] |
| Attempt/failure log | Avoid repeated failed paths | [...] | [...] | [...] |
| Verification evidence | Support pass claims | [...] | [...] | [...] |
| Handoff | Resume next session | [...] | [...] | [...] |

Use project-appropriate storage.

Do not assume a markdown file automatically creates persistence; the builder must actually write it.

---

## 12.2 Minimum Resume State

A fresh executor must be able to recover:

```text
CURRENT_CONTRACT_VERSION
CURRENT_TASK
TASK_STATE
LAST_TRUSTWORTHY_CHECKPOINT
LAST_VERIFICATION_RESULTS
KNOWN_FAILURES
BLOCKERS
REMAINING_RETRY_BUDGET
NEXT_PERMITTED_ACTION
```

---

# 13. Checkpoint and Handoff Contract

At every planned session boundary, record:

```text
SESSION/HANDOFF_ID:
[...]

CONTRACT_VERSION:
[...]

ACTIVE_TASK:
[...]

STATE:
[...]

WORK_ATTEMPTED:
[...]

CHANGES MADE:
[...]

VERIFICATION RUN:
[...]

PASSED:
[...]

FAILED:
[...]

BLOCKERS:
[...]

CHECKPOINT:
[...]

KNOWN RISKS:
[...]

NEXT PERMITTED ACTION:
[...]

EVIDENCE REFERENCES:
[...]
```

A blocked task may still have a clean handoff.

“Clean” means honest and reconstructable, not necessarily passing.

---

# 14. Rule Enforcement Register

Use this only for consequential controls.

Do not enumerate trivial stylistic guidance.

| Rule/requirement | Failure mode prevented | Applies when | Enforcement mechanism | Evidence of compliance | Violation response |
|---|---|---|---|---|---|
| [...] | [...] | [...] | [...] | [...] | [...] |

If no mechanism exists, write:

```text
ENFORCEMENT:
advisory only
```

Do not disguise advisory instructions as hard controls.

---

# 15. Change and Invalidation Rules

Define changes that invalidate previously accepted evidence.

| Change | Evidence invalidated | Required response |
|---|---|---|
| Relevant implementation changes | affected behavioral checks | rerun |
| Shared interface changes | dependent integration evidence | rerun |
| Acceptance criteria change | previous criterion evidence | reassess |
| Verification tool change | results produced by old mechanism if material | reassess/rerun |
| Environment change | environment-sensitive evidence | rerun |
| External authority changes | dependent assumptions/decisions | revisit |

Add project-specific rules.

---

# 16. Completion Definition

The project or requested unit of work is complete only when:

```text
[ ] All required acceptance criteria have valid evidence.
[ ] Required integration/end-to-end behavior has been verified.
[ ] Required approvals are distinct from and complete alongside technical checks.
[ ] No unresolved blocker contradicts the completion claim.
[ ] Verification applies to the current artifact/version.
[ ] Required state and handoff records are current.
[ ] Known limitations are disclosed.
```

Completion is not established by:

```text
code exists
agent confidence
self-review alone
a proposed test that was not run
a document saying verification should happen
an old passing result invalidated by later changes
```

---

# 17. Known Limitations and Accepted Risk

Not every risk requires more mechanism.

Record deliberately accepted limitations:

| ID | Limitation/risk | Reason accepted | Owner | Revisit trigger |
|---|---|---|---|---|
| R1 | [...] | [...] | [...] | [...] |

This prevents known gaps from being mistaken for omissions.

---

# 18. Remaining Gaps Before Execution

Separate blockers from non-blocking gaps.

## Blocking

```text
- G1: [...]
  Why it blocks:
  [...]
  Required resolution:
  [...]
```

## Non-Blocking

```text
- G2: [...]
  Risk:
  [...]
  Planned handling:
  [...]
```

If there are no blocking gaps and the quality review passes, the contract may be marked `ready_for_builder`.

---

# 19. Complexity Justification

Use only when the design includes non-trivial orchestration.

## Single Loop

```text
WHY A SINGLE LOOP IS SUFFICIENT:
[...]
```

or:

## Graph / Multi-Agent Structure

```text
WHY MULTIPLE NODES/AGENTS ARE NEEDED:
[...]

INDEPENDENT WORK UNITS:
[...]

BRANCHES/ROLLBACKS:
[...]

SHARED STATE:
[...]

ROUTING:
[...]

INTEGRATION:
[...]

COORDINATION COST:
[...]

WHY BENEFIT EXCEEDS COST:
[...]
```

Do not introduce a graph merely because several steps exist.

---

# 20. Builder Entry Instruction

End each completed execution contract with a short entry instruction.

Recommended form:

```text
Read this execution contract and the sources listed under Required Context.

Do not start implementation until the entry conditions for the first task are satisfied.

Work only on the currently active bounded task.

Use the specified evaluator and evidence requirements to determine whether it passes.

Classify failures before repair, stay within the retry budget, and do not silently expand scope.

If blocked, record the exact blocker and required decision.

Before ending a session, leave durable state showing the current task, evidence, failures, checkpoint, and next permitted action.
```

Adjust only where project-specific needs require it.

---

# 21. Quality Review

Before handoff to a builder, evaluate this contract using `QUALITY_RUBRIC.md`.

Record:

```text
QUALITY_REVIEW:
[pass | revise | blocked]

CRITICAL_FINDINGS:
- [...]

REPAIRS_MADE:
- [...]

REMAINING_EXCEPTIONS:
- [...]
```

Do not mark the contract `ready_for_builder` merely because all template sections contain text.

---

# 22. Small Worked Example

## Rough Idea

> “Add password reset.”

## Contract Fragment

```text
OBJECTIVE:
An existing user who has access to their registered email can request and complete a password reset.

IN_SCOPE:
- Request-reset behavior.
- Reset-token validation.
- New-password submission.
- Required tests.

OUT_OF_SCOPE:
- Account registration.
- OAuth.
- General email-system refactor.
- Production deployment.

FACTS:
F1 Existing users already authenticate with email/password.
Source: [AUTH_SPEC]

UNKNOWN:
U1 Whether a mail-capture service exists in the test environment.
Impact: End-to-end verification strategy.
Blocks: task activation until verification route is established.

TASKS:

T01 Establish password-reset test readiness.
State: not_started.

T02 Implement request-reset behavior.
Dependency: T01.
State: not_started.

T03 Implement reset-token/new-password behavior.
Dependency: T02.
State: not_started.

WIP:
1 active implementation task.

T02 ACCEPTANCE:

AC1:
Known account receives the expected reset-request outcome.
Evaluator: integration test.
Evidence: passing result.

AC2:
Unknown account behavior matches [AUTH_SPEC].
Evaluator: integration test.
Evidence: passing result.

LOOP T02:

Action:
Implement only request-reset behavior.

Verification:
Run [RESET_REQUEST_TEST].

Failure:
Classify before repair.

Retry budget:
3 implementation attempts.

Success:
All T02 acceptance checks pass.

Blocked:
Email test dependency unavailable or requirement conflict discovered.

Recovery:
Return to T02 task-start checkpoint.

INVALIDATION:
Changes to mail interface or auth user lookup require T02 re-verification.
```

Notice what this example does **not** claim:

- that the email system exists;
- that tests have passed;
- that three retries are universally correct;
- that the feature is already implemented.

It turns the idea into a bounded, testable execution structure while preserving uncertainty honestly.

---

# 23. Completion Standard for the Contract Itself

A fresh builder should be able to answer, without hidden conversation history:

1. What outcome am I trying to create?
2. What is explicitly outside scope?
3. Which facts are authoritative?
4. What assumptions or unknowns remain?
5. Is the environment ready?
6. Which task may I work on now?
7. What actions am I allowed to take?
8. What evidence proves success?
9. What should I do when verification fails?
10. How many repair attempts are permitted?
11. When must I stop or escalate?
12. How do I recover from failure?
13. What must I persist before handing off?
14. What later changes invalidate existing evidence?

If the builder cannot answer one of these and the answer materially affects execution, the contract is not yet complete.

---

# Project Addition: Contract Change Record

For long-running projects, optionally append a lightweight change record:

| Version | Change | Reason | Evidence invalidated? |
|---|---|---|---|
| [...] | [...] | [...] | yes/no + details |

This is not required for every small task.

Use it when contract changes themselves need to be auditable across sessions.
