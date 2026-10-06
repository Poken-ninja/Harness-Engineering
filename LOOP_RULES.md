# LOOP_RULES.md

## Purpose

Define reusable execution control for bounded tasks: task states and transitions, verification, failure handling, retry accounting, stopping, escalation, invalidation, cancellation, and recovery.

Context, authority, readiness, permissions, scope, persistence, and handoff live in `HARNESS_RULES.md`.

## Rule Metadata

When useful, record separately:

- **Strength:** principle, default, or project-dependent.
- **Provenance:** supplied material, design clarification, or project addition.

## 1. Task State Model

Primary states:

```text
not_started
active
blocked
passing
cancelled
superseded
```

Verification validity is tracked separately:

```text
not_verified
valid
stale
failed
```

A task state and verification status must not be conflated.

### Permitted transitions

| From | To | Guard | Required evidence |
|---|---|---|---|
| not_started | active | entry conditions pass; WIP slot available | activation record + prerequisite evidence |
| not_started | blocked | required prerequisite/decision is missing before execution | blocker record |
| not_started | cancelled | task is intentionally abandoned | cancellation decision |
| not_started | superseded | another task/contract version replaces it | supersession link |
| active | passing | all current acceptance criteria have valid evidence | completion evidence |
| active | blocked | execution cannot continue safely/productively within authority/budget | blocker record |
| active | cancelled | authorized owner stops the task | cancellation decision + preserved state |
| active | superseded | authorized contract/task replacement is adopted | supersession record |
| blocked | active | blocker resolved; entry conditions rechecked; WIP slot available | resolution evidence + activation record |
| blocked | cancelled | authorized owner abandons task | cancellation decision |
| blocked | superseded | replacement task/contract adopted | supersession record |
| passing | cancelled | task is retired and no longer required | cancellation/retirement decision |
| passing | superseded | replacement task/contract adopted | supersession record |

Direct `passing -> active` is **not automatic** when evidence becomes stale. See re-verification rules below.

## 2. Activation and Blocking

### LR-01 — Activation requires a guard check
**Strength:** principle  
**Provenance:** supplied material + design clarification

A task may enter `active` only when:
- its contract is defined enough to execute;
- required dependencies/context/permissions are available;
- its verification path is available or the task is explicitly an initialization task that establishes it;
- WIP capacity is available;
- required authorization is already valid or newly obtained.

### LR-02 — Pre-execution blockers use `not_started -> blocked`
**Strength:** principle  
**Provenance:** design clarification

If a task cannot start because of a missing dependency, decision, capability, or permission, record it as `blocked` rather than pretending execution began.

### LR-03 — Blocked resumption is explicit
**Strength:** principle  
**Provenance:** design clarification

A blocked task resumes only when:
1. the recorded blocker is resolved;
2. affected assumptions/readiness are rechecked;
3. WIP capacity is available;
4. the task is explicitly selected for activation.

Resolution of a blocker does not auto-activate the task.

## 3. Core Action–Verification–Repair Loop

### LR-04 — Every attempt is followed by verification
**Strength:** principle  
**Provenance:** supplied material

Core control flow:

```text
activate
  -> attempt
  -> verify
     -> pass
     -> classify failure
        -> repair
        -> diagnose
        -> block/escalate
        -> controlled stop
```

Do not accumulate unrelated unverified changes between significant attempts.

### LR-05 — Producer does not control the pass transition
**Strength:** principle  
**Provenance:** supplied material

The implementer may report readiness for verification, but `active -> passing` is controlled by the designated evaluator and required evidence.

## 4. Verification Integrity

### LR-06 — Acceptance criteria are not weakened to obtain a pass
**Strength:** principle  
**Provenance:** design clarification

Do not:
- weaken an acceptance criterion;
- delete or disable a failing check;
- change expected results to match defective behavior;
- narrow a test solely because the implementation currently fails.

A legitimate requirement change must come from the authorized requirement source or decision owner and creates a contract/specification change.

### LR-07 — Defective verification may be corrected only against authority
**Strength:** principle  
**Provenance:** design clarification

A test/check may be changed when evidence shows the verifier is wrong, flaky, obsolete, or inconsistent with the authoritative requirement.

Required record:

```text
verification defect
authoritative requirement
why the old check was wrong
change made
new verifier evidence
impact on prior results
```

Changing a verifier does not automatically validate the implementation; the corrected verifier must be run.

### LR-08 — Required behavior and observed behavior must be reconciled, not conflated
**Strength:** principle  
**Provenance:** design clarification

A failing runtime observation may indicate:
- implementation defect;
- stale/incorrect requirement;
- intentional version difference;
- environment/configuration difference;
- verification defect;
- unknown cause.

Classify before selecting a repair.

## 5. Failure Classification

Use only applicable classes:

```text
implementation_defect
verification_defect
context_gap
requirement_ambiguity
dependency_failure
environment_failure
permission_failure
scope_mismatch
architecture_conflict
resource_exhaustion
unknown_failure
```

### LR-09 — Classify before repeated repair
**Strength:** principle  
**Provenance:** project addition

After the first obvious correction, further retries require a recorded failure class and a material reason the next attempt may differ.

## 6. Retry Accounting

### LR-10 — The initial implementation attempt counts
**Strength:** principle  
**Provenance:** design clarification

If `ATTEMPT_LIMIT = N`, the first implementation attempt is attempt 1. The limit is the total number of implementation attempts for that task version.

Diagnostic observations do not count as implementation attempts unless they modify the candidate solution materially.

### LR-11 — Counters persist across sessions
**Strength:** principle  
**Provenance:** design clarification

Session restart, context reset, mode switching, or agent replacement does not reset attempt, diagnosis, time, cost, or risk budgets.

### LR-12 — Counters reset only for a materially new task version
**Strength:** principle  
**Provenance:** design clarification

A new budget may be created only when an authorized contract revision materially changes the objective, acceptance criteria, architecture approach, or task boundary enough to constitute a new version.

Preserve prior attempt history. Do not rename the same failing work merely to obtain a fresh budget.

### LR-13 — Extra budget requires explicit authority
**Strength:** principle  
**Provenance:** design clarification

The contract identifies who may authorize additional attempts/resources. The builder cannot self-extend its budget.

Record:
- previous budget;
- reason for extension;
- approving authority;
- new total ceiling.

### LR-14 — Diagnosis is separately bounded
**Strength:** principle  
**Provenance:** project addition

Unknown failures may enter diagnosis mode:

```text
observe -> hypothesis -> discriminating check -> update failure class
```

Diagnosis has its own bounded count/time/cost budget and also consumes the task's overall resource ceiling.

Switching into diagnosis does not bypass overall limits.

### LR-15 — Every task has an overall resource ceiling
**Strength:** principle  
**Provenance:** design clarification

Where material, define a total ceiling across implementation, diagnosis, verification, and recovery, using one or more of:
- attempts;
- elapsed time;
- inference/tool cost;
- externally visible operations;
- risk exposure.

## 7. Stop, Block, and Escalation

### LR-16 — Every loop has success, blocked, and controlled-stop exits
**Strength:** principle  
**Provenance:** supplied material

Success: all current acceptance criteria have valid evidence.  
Blocked: continuation requires an external dependency, decision, permission, or unavailable capability.  
Controlled stop: continuing would exceed budget, authority, or acceptable risk.

### LR-17 — Escalation is actionable
**Strength:** principle  
**Provenance:** supplied material

Record:
```text
blocker
why execution cannot continue
decision/input required
owner
resume condition
fallback if unresolved
```

## 8. Verification Invalidation and Re-verification

### LR-18 — Historical pass and current validity are separate
**Strength:** principle  
**Provenance:** design clarification

The supplied course describes `active -> passing` as irreversible. This framework preserves the historical fact that the task passed for a specific artifact/version, while tracking verification validity separately.

If a relevant later change occurs:

```text
task state: passing
verification: stale
attention_needed: true
```

Do not erase the old passing evidence.

### LR-19 — Stale evidence does not auto-activate work
**Strength:** principle  
**Provenance:** design clarification

When verification becomes stale:
- mark the affected evidence stale;
- identify the invalidation trigger and impact radius;
- queue re-verification or repair;
- preserve current WIP limits.

The task becomes `active` only when explicitly selected and a WIP slot is available.

### LR-20 — Re-verification may restore validity without implementation changes
**Strength:** principle  
**Provenance:** design clarification

If stale evidence is rerun against the current artifact and all required checks pass:

```text
task state remains passing
verification: stale -> valid
```

No implementation retry is consumed if no implementation modification occurred.

If re-verification fails, classify the failure. The task remains historically `passing` but current verification becomes `failed`; repair work must be explicitly activated under WIP limits.

## 9. Cancellation and Supersession

### LR-21 — Cancellation is explicit
**Strength:** principle  
**Provenance:** design clarification

Use `cancelled` when an authorized owner intentionally stops pursuing the task without replacing it.

Preserve:
- reason;
- partial work disposition;
- evidence/history;
- downstream impact.

Cancelled work is not passing.

### LR-22 — Supersession links old and new work
**Strength:** principle  
**Provenance:** design clarification

Use `superseded` when a new task or contract version replaces the old one.

Record:
- replacement ID/version;
- what carries forward;
- what evidence remains valid;
- what must be re-verified.

Supersession does not silently reset retry history for unchanged work.

## 10. Checkpoint and Recovery

### LR-23 — Checkpoint at meaningful recovery boundaries
**Strength:** principle  
**Provenance:** supplied material

Checkpoint before high-risk changes and after coherent trustworthy milestones when losing work would materially increase recovery cost.

### LR-24 — Recover to the smallest trustworthy boundary
**Strength:** principle  
**Provenance:** supplied material

Preferred order:

```text
repair current step
-> restore task checkpoint
-> restore task-start checkpoint
-> broader rollback
```

Preserve unrelated verified work.

### LR-25 — Recovery must be re-verified
**Strength:** principle  
**Provenance:** supplied material

Restoration is not proof of health. Rerun checks invalidated by the recovery before resuming normal execution.

## Compact Task Loop Contract

```text
TASK_ID:
STATE:
VERIFICATION_STATUS:

ENTRY_GUARD:
PERMITTED_ACTION:
ACCEPTANCE_CRITERIA:
EVALUATOR / PASSING_GATE:

FAILURE_CLASSES:
REPAIR_ROUTE:
DIAGNOSIS_ROUTE:

ATTEMPT_LIMIT:
DIAGNOSIS_BUDGET:
OVERALL_RESOURCE_CEILING:
BUDGET_EXTENSION_AUTHORITY:

BLOCK_CONDITIONS:
ESCALATION_OWNER:
STOP_CONDITIONS:

CHECKPOINT:
RECOVERY:
INVALIDATION_TRIGGERS:

CANCELLATION_AUTHORITY:
SUPERSESSION_RULE:
```

Use only the fields material to the task; do not create ceremony for trivial work.
