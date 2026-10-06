# KNOWLEDGE_MAP.md

## Purpose

This is the routing page for the Project's reusable knowledge.

Use it to decide **which source to consult next**. Keep detailed operating rules in their owning documents.

## Source Map

| Source | Contains | Consult when |
|---|---|---|
| **Project instructions** | Role, operating boundaries, required deliverables, and project-level policy | Always; these govern the reusable documents |
| **KNOWLEDGE_MAP.md** | Routing only | You need to know which source owns a question |
| **HARNESS_RULES.md** | Context selection, source authority, readiness, tools and permissions, scope/WIP, persistence, and handoffs | Designing the environment and boundaries around future execution |
| **LOOP_RULES.md** | Task states and transitions, verification, failure classification, retry accounting, stopping, escalation, invalidation, and recovery | Designing what happens after a bounded task enters execution |
| **EXECUTION_CONTRACT_TEMPLATE.md** | Compact default contract plus expanded sections for higher-risk or more complex work | Turning a new idea, prompt, or plan into a builder-ready specification |
| **QUALITY_RUBRIC.md** | Contract review checks for concreteness, consistency, verifiability, recoverability, and unnecessary complexity | Reviewing a draft contract before builder handoff |
| **Project-specific sources** | Requirements, architecture, repository facts, decisions, current state, and execution evidence | A future contract needs facts about the actual project |
| **External/live sources** | Current runtime state, CI, vendor docs, regulations, production signals, or other changing facts | Correctness depends on information that can change outside static project documents |

## Routing Rules

- **Source authority and source conflicts:** use `HARNESS_RULES.md`.
- **Required behavior versus observed runtime behavior:** use `HARNESS_RULES.md`; an observation is evidence, not automatically the governing requirement.
- **Contract readiness, initialization readiness, and feature readiness:** use `HARNESS_RULES.md` and represent the project-specific result in the execution contract.
- **Task activation, blocking, resumption, passing, evidence invalidation, re-verification, cancellation, or supersession:** use `LOOP_RULES.md`.
- **Retry counts, diagnosis budgets, resource ceilings, and escalation:** use `LOOP_RULES.md`.
- **Authorization, technical verification, and operational safeguards:** use `HARNESS_RULES.md`; task-level consequences are then referenced by `LOOP_RULES.md`.
- **What a future contract should contain:** use `EXECUTION_CONTRACT_TEMPLATE.md`.
- **Whether that contract is good enough to hand to a builder:** use `QUALITY_RUBRIC.md`.

## Information Types

Future contracts should distinguish:

- **fact** — supported by an identified source;
- **assumption** — a reversible belief used because information is missing;
- **decision** — an explicit choice by an authorized owner;
- **unknown** — unresolved information that may affect the design;
- **required behavior** — what the governing requirement says should happen;
- **observed behavior** — what a runtime, test, or other observation shows actually happened.

Detailed handling belongs in the owning rule or template document.

## Rule Metadata

When rule metadata is useful, keep these dimensions separate:

- **Strength:** principle, default, or project-dependent choice.
- **Provenance:** supplied material, design clarification, or project addition.

Provenance explains where a rule came from; it does not determine how strong the rule is.

## Standard Use

For a new idea or existing prompt:

1. Apply the **Project instructions**.
2. Use **HARNESS_RULES.md** to establish context, authority, readiness, permissions, scope, state, and handoff requirements.
3. Use **LOOP_RULES.md** only for tasks that can execute, retry, block, recover, or change state.
4. Fill the **compact execution contract** by default; expand only when uncertainty, risk, dependencies, or coordination justify it.
5. Review the result with **QUALITY_RUBRIC.md**.
6. Keep actual execution evidence in project-specific state/evidence sources, not in these reusable documents.

## Boundary

This file must remain short. If a rule needs an enforcement mechanism, evidence requirement, retry policy, state-transition guard, or violation response, it belongs in another document.
