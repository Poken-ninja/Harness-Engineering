# HARNESS_RULES.md

## Purpose

Define the reusable rules around future execution: context, authority, readiness, permissions, scope, persistent state, and handoff. Task lifecycle, retries, verification transitions, and recovery belong in `LOOP_RULES.md`.

## Rule Metadata

When metadata is useful, keep these dimensions separate:

- **Strength:** `principle`, `default`, or `project-dependent`.
- **Provenance:** `supplied material`, `design clarification`, or `project addition`.

Provenance describes origin; it does not determine strength.

For consequential rules, record: requirement, failure mode, applicability, enforcement mechanism, evidence, and violation response. If no enforcement mechanism exists, label the rule advisory.

## 1. Context and Source Authority

### HR-01 — Use the smallest sufficient context
**Strength:** principle  
**Provenance:** supplied material

Give a builder only the sources needed for the current task plus clearly routed on-demand sources.

- **Failure mode:** context dilution, stale guidance, excessive discovery cost.
- **Enforcement:** the contract names required context and on-demand context.
- **Evidence:** each required source maps to a constraint, dependency, acceptance criterion, verification step, or recovery need.
- **Violation response:** remove irrelevant context or justify it.

### HR-02 — Assign authority by domain
**Strength:** principle  
**Provenance:** design clarification

Do not declare one source globally authoritative. Name the governing source for each consequential domain, for example:

```text
required product behavior -> [PRODUCT_SPEC]
current implementation    -> [REPOSITORY]
observed runtime behavior -> [TARGET_ENVIRONMENT]
current CI result         -> [CI_SYSTEM]
external rule             -> [OFFICIAL_EXTERNAL_SOURCE]
```

Conflicting authoritative sources remain an explicit unresolved conflict until an authorized decision or source correction resolves them.

### HR-03 — Required behavior and observed behavior are different
**Strength:** principle  
**Provenance:** design clarification

A runtime observation reports what happened. It does not by itself decide what should happen.

When required and observed behavior differ, classify the discrepancy before changing code or documentation:

```text
implementation_defect
stale_or_incorrect_specification
intentional_version_difference
environment_or_configuration_difference
verification_defect
unknown
```

Resolve the classification using the domain authority map. Do not automatically declare either the implementation or documentation wrong.

### HR-04 — Refresh mutable sources when material
**Strength:** principle  
**Provenance:** supplied material

For changing sources, define a freshness rule or invalidation trigger when stale information could alter execution. If freshness cannot be established and the uncertainty is consequential, block the dependent task, not the entire contract.

## 2. Readiness

Readiness has three separate meanings.

### HR-05 — Contract readiness
**Strength:** principle  
**Provenance:** design clarification

A contract is **contract-ready** when a future builder can determine the objective, boundaries, source authority, unresolved gaps, initialization path if needed, first executable task, verification method, budgets, and handoff rules without inventing consequential control logic.

Runtime evidence is not required merely to write a usable contract. Unknown or unverified capabilities may remain if the contract provides a bounded way to establish them.

### HR-06 — Initialization readiness
**Strength:** principle  
**Provenance:** design clarification

An initialization task may start when its **own minimum prerequisites** exist. It must not require the capabilities it is supposed to create.

Typical minimum prerequisites are only those needed to perform setup safely, such as repository access, a supported shell/runtime installer, required credentials, or permission to create local configuration.

Initialization may establish capabilities such as:

```text
project starts
tests can run
logs are observable
state can persist
verification tools are installed
```

Its acceptance evidence proves initialization readiness for dependent work.

### HR-07 — Feature readiness
**Strength:** principle  
**Provenance:** supplied material + design clarification

A feature task may activate only when its task-specific dependencies, required context, permissions, and verification path are available. If initialization is required first, the feature remains not started or blocked according to `LOOP_RULES.md`.

### HR-08 — Never invent capability availability
**Strength:** principle  
**Provenance:** supplied material

Use capability status:

```text
available
unverified
unavailable
not_required
```

A contract may still be contract-ready with `unverified` capabilities if an initialization or discovery task can establish them.

## 3. Authorization, Verification, and Safeguards

### HR-09 — Keep the three concepts separate
**Strength:** principle  
**Provenance:** design clarification

- **Authorization:** permission to perform an action.
- **Technical verification:** evidence that the action produced the required result.
- **Operational safeguard:** a mechanism that reduces execution risk, such as a preview, dry run, backup, protected branch, staged rollout, or transaction boundary.

A dry run or preview is evidence/safeguard, not authorization. A successful test is verification, not authorization.

### HR-10 — Respect existing authorization within scope
**Strength:** principle  
**Provenance:** design clarification

Do not require repeated approval merely because an action is consequential if valid authorization already covers that action and remains current.

Re-authorization is required when the existing authorization is absent, expired, revoked, ambiguous, or exceeded by a material scope/risk change.

Record authorization scope where consequential.

### HR-11 — Separate read, write, and destructive capability
**Strength:** default  
**Provenance:** supplied material

Do not infer write permission from read permission, or destructive permission from ordinary write permission.

If technical enforcement is unavailable, describe the boundary as advisory rather than guaranteed.

## 4. Scope and WIP

### HR-12 — Every executable task is bounded
**Strength:** principle  
**Provenance:** supplied material

A task must have: ID, objective, scope, dependencies, acceptance criteria, verification method, and state. Explicit exclusions are required when adjacent scope creep is plausible.

Incidental discoveries are queued unless they block the active task or an authorized contract change brings them into scope.

### HR-13 — Default to one active implementation task
**Strength:** default  
**Provenance:** supplied material

Use WIP=1 unless parallel work is justified by independent scope, isolation, state ownership, integration rules, and post-integration verification.

A previously passing task whose evidence becomes stale **does not automatically become active**. It becomes attention-needed under `LOOP_RULES.md` and must wait for an available WIP slot before re-verification or repair.

### HR-14 — Parallel work must earn its coordination cost
**Strength:** principle  
**Provenance:** supplied material

Parallel tasks require:
- independent or explicitly coordinated scope;
- collision avoidance;
- owned state;
- integration point;
- parent/integration verification.

If those conditions are missing, return to serial execution.

## 5. Persistent State and Handoff

### HR-15 — Conversation history is not durable project state
**Strength:** principle  
**Provenance:** supplied material

Anything required to resume after a fresh session must live in a durable project artifact or external state system.

At minimum, preserve when applicable:

```text
contract version
task states
verification validity/evidence
decisions
known failures
budget usage
blockers
next permitted action
```

### HR-16 — Separate specification, status, and evidence
**Strength:** principle  
**Provenance:** supplied material

Keep distinct:
- what is required;
- what state the task is in;
- what evidence has actually been observed.

A written test requirement is not a passing result. A proposed persistence mechanism is not proof that state was persisted.

### HR-17 — Consequential state changes are attributable
**Strength:** principle  
**Provenance:** project addition

Record the cause and evidence for task-state changes, verification invalidation, authorization changes, and major decisions when reconstructability would otherwise be lost.

### HR-18 — A clean handoff is honest and resumable
**Strength:** principle  
**Provenance:** supplied material

A handoff may be passing, blocked, stale, or awaiting a decision. “Clean” means a fresh builder can reconstruct reality without hidden chat history.

Record:
- current task/state;
- verification status and evidence;
- failed attempts/diagnoses that should not be repeated;
- blocker or pending decision;
- checkpoint;
- remaining budget;
- next permitted action.

## 6. Harness Complexity

### HR-19 — Every mechanism must address a concrete failure mode
**Strength:** principle  
**Provenance:** supplied material

Do not add files, agents, approvals, gates, or automation ceremonially. Prefer the smallest mechanism that makes the relevant control enforceable or observable.

### HR-20 — Prefer mechanized checks for objective invariants
**Strength:** default  
**Provenance:** supplied material

Use deterministic checks where they are cheaper and more reliable than repeated prose or model judgment. Do not force subjective criteria into weak binary proxies merely to make them automated.

## Minimal Harness Review

Before builder handoff, confirm:

```text
[ ] Required behavior and observed behavior are distinguished.
[ ] Consequential domains have named authority.
[ ] Contract readiness is judged separately from runtime readiness.
[ ] Initialization tasks require only their own prerequisites.
[ ] Feature readiness is explicit.
[ ] Authorization, verification, and safeguards are not conflated.
[ ] Scope and WIP are bounded.
[ ] Stale prior evidence does not auto-activate work.
[ ] Durable resume state exists or is explicitly specified.
[ ] Advisory rules are not described as enforced.
[ ] Added mechanism addresses a concrete failure mode.
```
