# Coding Agent Skill Integration

## Intended pipeline

```text
User request
    ↓
Requirement Intent Gate
    ├─ PROCEED
    │    ↓
    │  Acquire necessary project/code information
    │    ↓
    │  Architecture Pseudocode Skill (when required)
    │    ↓
    │  Change Planning
    │    ↓
    │  Check whether proposed scope crosses module boundaries
    │    ├─ crossing not approved → CONFIRM
    │    └─ within boundary / approved
    │          ↓
    │      Scope-Locked Code Editing
    │          ↓
    │      Verification
    │          ↓
    │      Scope audit
    │          ↓
    │      Architecture maintenance agent, when required
    │
    ├─ CONFIRM → ask bounded decision, do not edit
    ├─ REPLAN  → revise requirement/scope
    └─ STOP    → do not execute
```

## Minimum handoff: Requirement Intent Gate → Architecture Pseudocode

```yaml
task:
  id: TASK-001
  description: ...
  requirement_contract: ...

project_context:
  architecture_md: ./architecture.md
  memory_md: ./memory.md
  conventions: ...
```

## Minimum handoff: Architecture Pseudocode → Change Planning

```yaml
requirement_contract: ...

architecture_slice:
  id: ...
  flow: ...
  stable_interfaces: ...
  runtime_boundaries: ...
  possible_boundary_crossings: ...
  invariants: ...
  consistency_check: ...
```

## Hard execution rules

1. A `CONFIRM`, `REPLAN`, or `STOP` gate blocks editing.
2. Architecture Pseudocode does not modify code.
3. Change Planning must name allowed files, symbols, operations, and forbidden operations.
4. Editing may not exceed the approved scope.
5. A new module boundary requires confirmation even when the code change appears small.
6. After verification, architecture.md is updated only when architecture actually changed.
