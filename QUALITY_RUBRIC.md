# QUALITY_RUBRIC.md

## Purpose

Review an execution contract before builder handoff. This rubric evaluates the specification, not whether implementation has occurred.

Outcome:

```text
PASS
REVISE
BLOCKED
```

A contract can pass while runtime evidence is still pending if initialization and feature-readiness paths are explicit and bounded.

## 1. Hard-Fail Checks

A contract cannot pass if any applicable condition is true:

```text
[ ] No observable outcome is defined.
[ ] Required behavior has no identified authority where authority matters.
[ ] Required and observed behavior are conflated.
[ ] The first executable task has no bounded scope.
[ ] A consequential acceptance criterion has no evaluator/evidence path.
[ ] A repeatable loop has no finite attempt/diagnosis/resource ceiling.
[ ] Retry counters can reset merely by changing session, agent, or mode.
[ ] The builder can weaken criteria or verification solely to obtain a pass.
[ ] A required authorization boundary is missing.
[ ] Authorization, verification, and safeguards are treated as interchangeable.
[ ] A high-impact recovery path requires unauthorized destructive action.
[ ] A claim is described as enforced or verified without mechanism/evidence.
```

## 2. Objective and Scope

Check:

```text
[ ] Outcome is behavioral and observable.
[ ] In-scope and likely out-of-scope adjacent work are clear.
[ ] Incidental discoveries have a route.
[ ] Repair cannot silently expand objective, permissions, or acceptance criteria.
[ ] WIP is bounded.
```

Default WIP=1 is acceptable but not mandatory when justified isolation and integration exist.

## 3. Knowledge and Authority

Check:

```text
[ ] Facts, assumptions, decisions, and unknowns are distinguished.
[ ] Domain authority is explicit for consequential claims.
[ ] Mutable sources have freshness/invalidation rules where material.
[ ] Required behavior is separate from observed runtime behavior.
[ ] A mismatch can be classified as implementation defect, stale spec,
    intentional version difference, environment difference, verifier defect, or unknown.
```

Do not automatically prefer runtime observation over specification, or specification over runtime observation, without resolving the domain question.

## 4. Readiness Separation

### Contract readiness
Can a future builder understand objective, boundaries, authority, unresolved gaps, the next task, verification, budgets, and handoff without inventing consequential control logic?

Runtime evidence is not required for contract readiness if the contract correctly marks it unverified and provides a bounded initialization/discovery path.

### Initialization readiness
If an initialization task exists:

```text
[ ] It requires only prerequisites necessary to perform setup safely.
[ ] It does not require the capabilities it exists to establish.
[ ] Its outputs have acceptance evidence.
```

### Feature readiness
For the feature task:

```text
[ ] Dependencies are satisfied.
[ ] Required context is available.
[ ] Required permission is valid.
[ ] Verification path is available.
[ ] WIP slot is available before activation.
```

## 5. Authorization, Verification, Safeguards

Check all three independently:

```text
AUTHORIZATION:
[ ] Existing authorization is respected within its scope.
[ ] Re-approval is required only for absent/expired/revoked/exceeded authority.

TECHNICAL VERIFICATION:
[ ] Required outcome has an evaluator and evidence.

SAFEGUARDS:
[ ] Relevant previews, backups, protected environments, or staged actions
    are treated as risk controls, not permission.
```

A dry run is not authorization. Approval is not technical verification.

## 6. Task Lifecycle Completeness

Confirm the contract is compatible with these transitions:

```text
not_started -> active
not_started -> blocked
blocked -> active
active -> passing
active -> blocked
* -> cancelled where authorized
* -> superseded where replaced
```

Check guards:

```text
[ ] Activation requires entry conditions + WIP slot.
[ ] Pre-execution missing prerequisites can block without pretending execution began.
[ ] Blocker resolution does not auto-activate the task.
[ ] Cancellation records disposition/history.
[ ] Supersession links old and replacement work.
```

Verification validity must be separate from lifecycle state.

## 7. Verification Integrity

Check:

```text
[ ] Acceptance criteria cannot be weakened just to pass.
[ ] Failing checks cannot be deleted/disabled merely to pass.
[ ] Expected results cannot be rewritten merely to match observed defects.
[ ] Legitimate verifier corrections cite the authoritative requirement.
[ ] Corrected checks are rerun.
[ ] Same-context self-review is not labeled independent.
```

Any contract that permits pass-seeking manipulation of its verifier is BLOCKED.

## 8. Retry Accounting

Check:

```text
[ ] Initial implementation attempt counts as attempt 1.
[ ] Implementation attempts are distinguished from non-modifying diagnosis.
[ ] Diagnosis has its own budget.
[ ] Overall resource use is bounded.
[ ] Budgets persist across sessions, agent swaps, and mode switches.
[ ] Budget resets require a materially new authorized task version.
[ ] Prior history is preserved after reset/version change.
[ ] Additional budget has a named authorizing owner.
```

Reject any mechanism that can bypass limits by relabeling work or restarting context.

## 9. Failure and Repair Logic

Check:

```text
[ ] Failures are classified before repeated repair.
[ ] A retry states what materially changed.
[ ] Unknown failures enter bounded diagnosis rather than random editing.
[ ] Success, blocked, and controlled-stop exits exist.
[ ] Escalation names the owner, required decision, and resume condition.
```

## 10. Verification Invalidation and Re-verification

Check:

```text
[ ] Historical pass evidence is preserved.
[ ] Current verification validity can become stale.
[ ] Staleness has an identifiable trigger/impact radius.
[ ] A stale passing task does not auto-activate.
[ ] Re-verification waits for a WIP slot when active work is required.
[ ] Passing re-verification restores validity without inventing an implementation attempt.
[ ] Failed re-verification is classified before repair.
```

This is a design clarification layered on top of the supplied material's irreversible pass-state description.

## 11. Recovery and Persistence

Check:

```text
[ ] Meaningful checkpoints exist where recovery cost/risk warrants them.
[ ] Recovery uses the smallest trustworthy boundary.
[ ] Recovery preserves unrelated valid work.
[ ] A restored state is re-verified where needed.
[ ] Failed approaches remain in durable history.
[ ] A fresh session can recover current state, validity, blockers, budgets,
    checkpoint, and next action.
```

## 12. Complexity Discipline

Check:

```text
[ ] Compact contract form is used when sufficient.
[ ] Expanded sections exist only because risk/uncertainty/dependency/coordination warrants them.
[ ] Task data is not needlessly repeated across sections.
[ ] Every added rule/gate/agent/file addresses a concrete failure mode.
[ ] Deterministic checks are preferred for objective criteria.
[ ] Graph/multi-agent structure is justified by control-flow needs.
[ ] Coordination/review cost is acknowledged.
```

The best contract is the smallest one that leaves consequential control explicit.

## 13. Final Fresh-Builder Test

A fresh builder should be able to answer:

```text
1. What outcome is required?
2. What is outside scope?
3. Which source defines required behavior?
4. What observed behavior is known, if any?
5. Is the contract ready even if runtime is not?
6. If setup is missing, what initialization task may run now?
7. Is the feature itself ready?
8. Which task may activate under WIP?
9. What authorization already exists?
10. What evidence proves success?
11. What verifier changes are forbidden?
12. How are retries counted?
13. What diagnosis/overall budget remains?
14. When must execution stop or escalate?
15. What happens if prior evidence becomes stale?
16. How is blocked work resumed?
17. How can work be cancelled or superseded?
18. What must survive handoff?
```

If the builder must invent a consequential answer, revise the contract.

## 14. Review Record

```text
OUTCOME: PASS | REVISE | BLOCKED

CRITICAL:
- [...]

MAJOR:
- [...]

MINOR:
- [...]

SIMPLIFICATIONS_MADE:
- [...]

CONTRADICTIONS_REPAIRED:
- [...]

ACCEPTED_EXCEPTIONS:
- [...]

FINAL_CONTRACT_READINESS:
ready | not_ready

NOTE:
This review evaluates the specification only. It is not proof that implementation,
initialization, verification, authorization, or runtime behavior has been executed.
```
