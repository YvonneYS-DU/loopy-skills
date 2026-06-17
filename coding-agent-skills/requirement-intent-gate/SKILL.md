---
name: requirement-intent-gate
description: >
  Before a coding agent plans or edits code, infer the user's actual intent from the current request,
  persistent user preferences, repository conventions, and prior decisions. Prevent the agent from
  silently choosing defaults or expanding the task. Produce an explicit requirement contract and
  decide whether execution may proceed, must pause for confirmation, or must present bounded options.
version: 1.0.0
---

# Requirement Intent Gate

## Purpose

This skill is not a generic requirement clarification questionnaire.

Its job is to:

1. Use known user habits and prior decisions as first-class context.
2. Distinguish what the user explicitly requested from what the agent merely inferred.
3. Prevent the coding agent from selecting an implementation direction by default.
4. Detect decisions that would change architecture, behavior, data contracts, dependencies, UI,
   model usage, regex logic, infrastructure, or other module boundaries.
5. Produce a stable `Requirement Contract` before change planning begins.

The output of this skill is a decision gate, not an implementation plan.

---

## When to invoke

Invoke before `Change Planning` when any of the following is true:

- A coding task has more than one reasonable implementation.
- The request contains words such as “优化”, “处理一下”, “支持”, “修复”, “改成”, “增加”,
  but the expected behavior is not fully specified.
- The task may alter a public interface, schema, prompt, model provider, regex, frontend behavior,
  database, queue, deployment configuration, or module responsibility.
- The agent is about to make a default choice that the user did not state.
- Existing memories or project decisions materially affect the interpretation.
- The user has previously rejected a common default.
- The requested change may exceed the current module boundary.

This skill may be skipped only when all conditions are true:

- The requested behavior is explicit.
- There is one obvious implementation path.
- No module boundary changes.
- No public behavior or contract changes.
- No irreversible or difficult-to-revert action.
- The change is small enough to be fully represented in a concise Requirement Contract.

Even when skipped interactively, a minimal Requirement Contract must still be recorded.

---

## Required context

Read, when available:

1. Current user request.
2. User preference memory.
3. Project `memory.md`.
4. Project `architecture.md`.
5. Existing task notes, ADRs, conventions, and previous accepted/rejected decisions.
6. Relevant code evidence only when necessary to resolve requirement meaning.

Do not read the entire repository merely to clarify intent.

---

## User preference principles

Treat persistent preferences as constraints, not decoration.

Examples:

- The user prefers explicit plans over autonomous implementation.
- The user does not want the agent to silently expand scope.
- The user values clean module boundaries.
- The user expects architecture knowledge to be preserved in project Markdown.
- The user may prefer a local change even when a broad refactor appears cleaner.
- Existing project conventions take precedence over generic “best practices”.

Never invent a preference. Classify every preference as:

- `confirmed`: explicitly stated or stored.
- `inferred`: strongly suggested by repeated behavior.
- `unknown`: not available.

Only `confirmed` preferences may act as hard constraints.
`inferred` preferences may trigger confirmation but cannot silently determine implementation.

---

## Decision model

Classify each requirement item into one of four groups.

### A. Explicit

Directly stated by the user.

Example:

```yaml
- id: R1
  statement: "Save architecture pseudocode in memory.md"
  source: current_request
  confidence: confirmed
```

### B. Inherited constraint

Comes from confirmed user preference, architecture, or an accepted prior decision.

```yaml
- id: C1
  statement: "Do not expand the approved modification scope automatically"
  source: user_preference
  confidence: confirmed
```

### C. Safe inference

Necessary implementation detail that:

- does not change external behavior,
- stays within an existing module boundary,
- is reversible,
- follows an established project convention.

Safe inferences may proceed, but must be recorded.

### D. Decision requiring confirmation

Confirmation is required when a choice changes one or more of:

- user-visible behavior,
- API or output schema,
- model/provider/prompt strategy,
- regex responsibility or matching semantics,
- frontend workflow,
- persistence or database structure,
- queue/async/cache behavior,
- dependency versions,
- deployment or infrastructure,
- security or secrets handling,
- module ownership,
- task scope,
- acceptance criteria.

---

## Boundary categories

Use these standard boundary labels:

```text
ai_model
prompt
regex
frontend
backend_api
domain_logic
data_schema
database
queue
async_runtime
cache
filesystem
external_service
deployment
security
tests
documentation
```

A task may involve multiple boundaries.

---

## Gate policy

Return exactly one gate decision:

### `PROCEED`

Use only when:

- intent is sufficiently determined,
- unresolved items are non-blocking,
- no unapproved boundary crossing exists.

### `CONFIRM`

Use when one or more material choices require the user to decide.

Do not ask broad questions. Present the smallest bounded decision possible:

```text
This change can remain inside `regex`, or move part of the decision into the AI extraction layer.

A. Keep regex as the only classifier.
B. Let AI classify uncertain cases after regex.
Recommended: B, because ...
```

Do not execute until confirmed.

### `REPLAN`

Use when the current task conflicts with a prior accepted requirement, architecture rule, or approved scope.

### `STOP`

Use for destructive, unsafe, unauthorized, or fundamentally contradictory actions.

---

## Output: Requirement Contract

Write or return the following structure:

```yaml
requirement_contract:
  task_id: string
  task_summary: string

  user_goal:
    primary: string
    secondary: []

  explicit_requirements:
    - id: R1
      statement: string
      evidence: string

  inherited_constraints:
    - id: C1
      statement: string
      source: user_memory | project_memory | architecture | prior_decision

  safe_inferences:
    - id: I1
      statement: string
      rationale: string

  prohibited_defaults:
    - id: D1
      agent_must_not_assume: string

  affected_boundaries:
    - boundary: regex
      expected_change: string
      status: inside_existing_boundary | possible_crossing | confirmed_crossing

  acceptance_criteria:
    - id: A1
      statement: string
      verification: string

  unresolved_decisions:
    - id: Q1
      decision: string
      options:
        - id: A
          description: string
          impact: string
        - id: B
          description: string
          impact: string
      recommendation:
        option: B
        reason: string
      blocking: true

  gate:
    decision: PROCEED | CONFIRM | REPLAN | STOP
    reason: string
```

---

## Response behavior

When `PROCEED`:

- Do not ask for confirmation.
- Pass the Requirement Contract to Change Planning.
- State any safe inference explicitly in the internal task record.

When `CONFIRM`:

- Do not write code.
- Do not produce a final change plan as if the decision were settled.
- Ask only about blocking choices.
- Prefer 2–3 bounded options.
- Include a recommendation, but do not treat it as approval.

When `REPLAN`:

- Identify the exact conflict.
- Preserve already confirmed requirements.
- Return the smallest required adjustment.

---

## Anti-patterns

Do not:

- Ask the user to restate information already stored in memory.
- Ask generic questions such as “Can you provide more details?”
- Choose a framework, provider, schema, or architecture by default.
- convert a local fix into a refactor.
- turn an implementation preference into a requirement.
- treat generic best practice as stronger than the user's confirmed convention.
- confuse “possible improvement” with requested scope.
- start repository-wide exploration before the requirement gate is resolved.

---

## Completion checklist

Before finishing, verify:

- [ ] Explicit requirements are separated from inferred ones.
- [ ] Confirmed user habits are applied.
- [ ] No user preference was invented.
- [ ] Prohibited defaults are listed.
- [ ] Module boundaries are identified.
- [ ] Blocking decisions are minimal and bounded.
- [ ] The gate decision is unambiguous.
- [ ] No implementation was started before the gate.
