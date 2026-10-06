# EXECUTION_CONTRACT_TEMPLATE.md

## Purpose

Use this template to turn a new idea, rough prompt, or existing plan into a builder-ready execution contract.

A contract may be ready before runtime evidence exists. It must make unknowns, initialization needs, feature-readiness conditions, verification, budgets, and handoff explicit. It does not prove that implementation or verification has occurred.

# 1. Default Compact Contract

Use this format unless uncertainty, risk, dependencies, multiple tasks, approvals, or coordination justify expansion.

```text
CONTRACT_ID:
PROJECT:
STATUS: draft | contract_ready | blocked

OBJECTIVE:
[observable outcome]

IN_SCOPE:
- [...]

OUT_OF_SCOPE:
- [...]

REQUIRED_BEHAVIOR:
- RB1: [...]

OBSERVED_BEHAVIOR:
- [only if known; otherwise "not yet observed"]

KEY_FACTS / ASSUMPTIONS / UNKNOWNS:
- Fact: [...]
- Assumption: [...]
- Unknown: [...]

SOURCE_AUTHORITY:
- Requirements -> [...]
- Current implementation -> [...]
- Runtime observation -> [...]

READINESS:
- Contract readiness: ready | not_ready
- Initialization readiness: ready | not_ready | not_applicable
- Feature readiness: ready | not_ready | not_yet_evaluated

NEXT TASK:
- ID:
- Type: initialization | feature | verification | recovery
- State: not_started | blocked | active | passing | cancelled | superseded
- Verification status: not_verified | valid | stale | failed
- Entry guard:
- Scope:
- Acceptance criteria:
- Verification:
- Retry/resource budget:
- Checkpoint/recovery:
- Block/escalation route:

AUTHORIZATION / SAFEGUARDS:
- Authorized actions:
- Approval required only for:
- Operational safeguards:

PERSISTENCE / HANDOFF:
- State location:
- Evidence location:
- Next-action record:

REMAINING BLOCKERS:
- [...]
```

If this compact form gives a fresh builder enough control to begin the next bounded task safely, stop here.

# 2. When to Expand

Expand only when one or more apply:

- multiple dependent tasks must be planned;
- initialization is non-trivial;
- requirements or source authority conflict;
- high-impact actions need explicit authorization boundaries;
- independent evaluation matters;
- retry/diagnosis/recovery logic is non-trivial;
- parallel work or graph coordination is justified;
- several acceptance criteria need separate evidence mapping;
- durable handoff across sessions is consequential.

Do not repeat the same task information in several sections. Reference task IDs instead.

# 3. Contract Metadata

```text
CONTRACT_ID:
PROJECT:
VERSION:
STATUS: draft | contract_ready | blocked
CREATED_FROM:
LAST_REVIEWED:
```

`contract_ready` means the specification can guide the builder. It does not mean initialization or feature execution is currently ready.

# 4. Objective and Boundaries

```text
PROBLEM:
[...]

USERS / BENEFICIARIES:
[...]

DESIRED OUTCOME:
[...]

SUCCESS CRITERIA:
- SC1: [...]
- SC2: [...]

IN_SCOPE:
- [...]

OUT_OF_SCOPE:
- [...]

CONSTRAINTS:
- [...]
```

Describe required behavior, not implementation preference unless the implementation choice is itself a constraint.

# 5. Facts, Assumptions, Decisions, Unknowns

| ID | Type | Statement | Source/owner | Impact / invalidation |
|---|---|---|---|---|
| F1 | fact | [...] | [...] | [...] |
| A1 | assumption | [...] | [...] | [...] |
| D1 | decision | [...] | [...] | [...] |
| U1 | unknown | [...] | [...] | [...] |

Do not fill unknowns with guesses merely to make the contract look complete.

# 6. Required vs Observed Behavior

| ID | Required behavior | Authority | Observed behavior | Observation source | Current interpretation |
|---|---|---|---|---|---|
| RB1 | [...] | [...] | [...] | [...] | matches / implementation defect / possible stale spec / intentional version difference / environment difference / unknown |

A runtime mismatch is evidence requiring classification, not automatic proof that either code or documentation is wrong.

# 7. Source and Context Map

| Domain | Authoritative source | Freshness/invalidation | On-demand supporting context |
|---|---|---|---|
| Requirements | [...] | [...] | [...] |
| Implementation | [...] | [...] | [...] |
| Runtime behavior | [...] | [...] | [...] |
| Verification status | [...] | [...] | [...] |

Required context for the next task:

```text
- [source]: [reason]
```

# 8. Readiness Model

## 8.1 Contract Readiness

```text
STATUS: ready | not_ready
MISSING CONTROL INFORMATION:
- [...]
```

Contract readiness is judged from specification quality, not from whether runtime checks have already been executed.

## 8.2 Initialization Readiness

Use when setup capability must be created.

```text
INITIALIZATION_REQUIRED: yes | no
INITIALIZATION_TASK_ID:
MINIMUM_PREREQUISITES:
- [...]
CAPABILITIES TO ESTABLISH:
- [...]
INITIALIZATION_ACCEPTANCE:
- [...]
```

Do not require an initialization task to already possess the capability it exists to establish.

## 8.3 Feature Readiness

For each candidate feature task:

```text
TASK_ID:
READY: yes | no | not_yet_evaluated
MISSING:
- dependencies
- context
- permission
- verification path
```

# 9. Authorization, Verification, Safeguards

Keep these separate.

```text
AUTHORIZATION:
- Existing authorized scope:
- New approval required for:

TECHNICAL VERIFICATION:
- Evaluator:
- Check:
- Passing evidence:

OPERATIONAL SAFEGUARDS:
- preview/dry run:
- checkpoint/backup:
- protected environment/staged rollout:
```

Do not treat a dry run as authorization or an approval as technical proof.

# 10. Task Plan

Use one row per meaningful task.

| Task | Type | Dependency | State | Verification status |
|---|---|---|---|---|
| T01 | initialization/feature/verification/recovery | [...] | not_started | not_verified |

Default active implementation WIP is 1 unless parallelism is justified in the contract.

A passing task with stale evidence remains `passing` and `stale`; it is not automatically active.

# 11. Task Contract

Create detailed task sections only for the next executable task and for later tasks whose constraints are already consequential.

## [TASK_ID] — [NAME]

```text
TYPE:
STATE:
VERIFICATION_STATUS:

OBJECTIVE:
IN_SCOPE:
OUT_OF_SCOPE:

DEPENDENCIES:
ENTRY_GUARD:

PERMITTED_ACTIONS:
PROHIBITED_ACTIONS:

ACCEPTANCE_CRITERIA:
- AC1: [...]

EVALUATOR / PASSING_GATE:
FAST_FEEDBACK_CHECKS:

INVALIDATION_TRIGGERS:

ATTEMPT_LIMIT:
DIAGNOSIS_BUDGET:
OVERALL_RESOURCE_CEILING:
BUDGET_EXTENSION_AUTHORITY:

BLOCK_CONDITIONS:
ESCALATION_OWNER:
CANCELLATION_AUTHORITY:
SUPERSESSION_RULE:

CHECKPOINT:
RECOVERY:
```

The initial implementation attempt counts as attempt 1. Budgets persist across sessions.

# 12. Verification Integrity

For each acceptance criterion, confirm:

```text
AUTHORITATIVE REQUIREMENT:
EVALUATOR:
EXPECTED RESULT:
EVIDENCE:
```

Rules:
- do not weaken criteria to obtain a pass;
- do not delete/disable failing checks solely to obtain a pass;
- do not change expected results merely to match current behavior;
- if a verifier is defective, justify its correction against the authoritative requirement and record the change;
- rerun corrected verification before claiming success.

# 13. Acceptance-to-Evidence Matrix

Use only when several criteria or tasks make the mapping non-obvious.

| Criterion | Task | Evaluator | Evidence | Invalidation trigger |
|---|---|---|---|---|
| SC1 | T02 | [...] | [...] | [...] |

# 14. Independent Evaluation

Include only when independence is materially required.

```text
WHY REQUIRED:
PRODUCER:
EVALUATOR:
CONTEXT ISOLATION:
SHARED INPUTS:
DISAGREEMENT ROUTE:
```

Same-agent same-context review is self-review.

# 15. Parallel / Graph Coordination

Include only when justified.

```text
WHY SERIAL EXECUTION IS INSUFFICIENT:
NODES / OWNERS:
SHARED STATE:
ROUTING:
ROLLBACK PATHS:
ISOLATION:
INTEGRATION GATE:
COORDINATION COST:
```

Multiple steps alone do not justify a graph.

# 16. Persistent State and Handoff

```text
STATE STORE:
DECISION STORE:
ATTEMPT / DIAGNOSIS LOG:
VERIFICATION EVIDENCE:
CHECKPOINT:
HANDOFF LOCATION:
```

A fresh builder should recover:

```text
contract version
current task/state
verification validity
blockers
attempt and diagnosis budgets used
overall budget remaining
last trustworthy checkpoint
next permitted action
```

# 17. Completion and Status Claims

A task may be `passing` only when current acceptance criteria have valid evidence.

If later evidence becomes stale:
- preserve historical passing evidence;
- mark verification `stale`;
- queue re-verification;
- do not auto-activate the task.

A contract may be `contract_ready` even when initialization or feature readiness is still pending, provided the path to establish readiness is bounded and clear.

# 18. Remaining Gaps

```text
BLOCKING FOR CONTRACT READINESS:
- [...]

BLOCKING FOR INITIALIZATION:
- [...]

BLOCKING FOR FEATURE EXECUTION:
- [...]

NON-BLOCKING / ACCEPTED RISK:
- [...]
```

# 19. Builder Entry Instruction

```text
Read this contract and its required sources.
Do not invent missing authority, capability, or evidence.
Activate work only through the defined task guard and WIP limit.
Preserve acceptance criteria and verification integrity.
Count the initial implementation attempt as attempt 1 and preserve budgets across sessions.
Classify failures before repeated repair.
If evidence becomes stale, queue re-verification rather than auto-activating old work.
Leave durable state showing evidence, budgets, blockers, checkpoint, and next permitted action.
```

# 20. Quality Review

Review with `QUALITY_RUBRIC.md`.

```text
OUTCOME: pass | revise | blocked
CRITICAL_FINDINGS:
REPAIRS:
ACCEPTED_EXCEPTIONS:
```
