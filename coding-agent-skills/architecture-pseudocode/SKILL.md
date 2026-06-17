---
name: architecture-pseudocode
description: >
  Build a task-scoped architecture representation from maintained project documentation and verified
  code. Express data flow, module responsibilities, stable interfaces, runtime boundaries, branches,
  state changes, and error propagation as architecture-level pseudocode. Persist the result in
  memory.md, check consistency with architecture.md, and request architecture.md synchronization
  after implementation when the architecture actually changes.
version: 1.0.0
---

# Architecture Pseudocode Skill

## Purpose

This skill replaces repository-wide structural narration and traditional code-search output.

It does not produce a file tree and does not merely report where symbols are located.

It constructs a task-scoped, implementation-useful architecture slice that explains:

- how data flows,
- how modules collaborate,
- where major branches occur,
- where state changes,
- where external services are called,
- how errors propagate,
- which interfaces are stable boundaries,
- which runtime boundaries exist,
- which paths can be affected by the proposed change.

The result must be concrete enough to support Change Planning.

---

## Invocation policy

This skill is not mandatory for every coding task.

### Invoke when

- More than one module participates.
- The change may cross a module boundary.
- The execution path is not obvious from the maintained documentation.
- The task involves AI, prompt, regex, frontend, backend, database, queue, async, cache,
  external service, deployment, or schema boundaries.
- A stable interface or data contract may be affected.
- The code and `architecture.md` may no longer agree.
- Change Planning cannot safely define scope without a local architecture slice.

### Skip when

- The change is trivial and contained in one known symbol.
- No boundary, flow, state, schema, or external behavior changes.
- The current `memory.md` already contains a fresh, verified architecture slice for the exact path.
- Change Planning can establish scope without additional architectural analysis.

When skipped, record:

```yaml
architecture_pseudocode:
  skipped: true
  reason: string
```

---

## Sources of truth

Use sources in this priority order:

1. Verified current code behavior.
2. Accepted architecture decisions.
3. Current `architecture.md`.
4. Current project `memory.md`.
5. Tests that demonstrate runtime behavior.
6. Inference.

If code conflicts with `architecture.md`, do not silently choose one.

Record the inconsistency:

```yaml
consistency_issue:
  architecture_md_says: ...
  code_does: ...
  impact: ...
  action: architecture_update_required | code_fix_required | needs_confirmation
```

---

## Required input

```yaml
task:
  description: string
  requirement_contract: object

project_context:
  architecture_md: path
  memory_md: path
  relevant_code:
    - path: string
      symbols: []
  conventions: []

optional:
  known_entrypoint: string
  suspected_modules: []
  existing_architecture_slice_id: string
```

---

## Information acquisition

Acquire only information necessary to explain the task path.

Preferred sequence:

```text
Read Requirement Contract
    ↓
Read relevant section of architecture.md
    ↓
Read relevant section of memory.md
    ↓
Identify known entrypoint and stable boundary
    ↓
Inspect only the code needed to verify the flow
    ↓
Inspect tests when behavior remains ambiguous
    ↓
Stop when Change Planning has enough evidence
```

Do not perform unlimited repository exploration.

Traditional search may be used internally to locate evidence, but its raw result is not the final output.

---

## Required runtime boundary labels

Mark relevant boundaries explicitly:

```text
transaction
async
queue
cache
database
filesystem
network
external_api
llm_provider
ocr_provider
subprocess
container
reverse_proxy
cloud_platform
frontend_backend
security
```

Each boundary should include:

- entering module,
- leaving module,
- data contract,
- failure mode,
- retry/timeout behavior when known,
- whether the task may alter it.

---

## Output structure

```yaml
architecture_slice:
  id: string
  task_id: string
  generated_at: string
  status: verified | partially_verified

  scope:
    entrypoint: string
    terminal_output: string
    included_modules: []
    excluded_modules: []

  flow:
    - step: 1
      module: string
      symbol: string
      responsibility: string
      input: string
      output: string
      next: []

  branches:
    - condition: string
      source: confirmed | inferred | unknown
      paths: []

  state_changes:
    - location: string
      before: string
      after: string

  stable_interfaces:
    - interface: string
      owner_module: string
      contract: string
      must_preserve: true

  runtime_boundaries:
    - type: async
      from: string
      to: string
      contract: string
      failure_behavior: string
      task_impact: none | possible | confirmed

  error_propagation:
    - origin: string
      transformed_by: []
      surfaced_at: string

  affected_paths:
    - path: string
      impact: direct | indirect | none
      reason: string

  evidence:
    confirmed: []
    inferred: []
    unknown: []

  consistency_check:
    architecture_md_consistent: true | false | partial
    issues: []

  change_planning_notes:
    safe_module_boundary: string
    possible_boundary_crossings: []
    invariants: []
```

---

## Architecture-level pseudocode format

The main output must include readable pseudocode.

Example:

```python
def process_document(file):
    # frontend_backend boundary
    validated_file = UploadController.validate(file)

    # domain entrypoint
    document = DocumentService.open(validated_file)

    if PDFDetector.is_searchable(document):
        page_content = TextExtractor.extract(document)
    else:
        # external_api + ocr_provider boundary
        page_content = OCRService.extract(document)

    # stable interface:
    # both branches must return PageContent[]
    sections = SectionClassifier.classify(page_content)

    results = []
    for section in sections:
        extractor = ExtractorRegistry.for_issuer(section.issuer)
        raw_result = extractor.extract(section)

        validation = SchemaValidator.validate(raw_result)

        if validation.ok:
            results.append(validation.data)
        else:
            repaired = RepairService.repair(
                raw_result,
                validation.errors,
            )
            results.append(repaired)

    return ResultMerger.merge(results)
```

Also include a concise flow representation when useful:

```text
UploadController
    ↓ File
DocumentService
    ↓ Document
PDFDetector
    ├─ searchable → TextExtractor
    └─ scanned    → OCRService [external_api, timeout]
                         ↓
                PageContent[]  ← stable interface
                         ↓
                SectionClassifier
                         ↓
                IssuerExtractor
                         ↓
                SchemaValidator
                         ↓
                ResultMerger
```

Mermaid may be included in the persisted memory record, but it is not mandatory for every task.

---

## Evidence labels

Every non-trivial statement must be classified:

### Confirmed

Verified in current code, test, or accepted architecture decision.

### Inferred

Strongly implied, but not directly verified.

### Unknown

Required information is missing or contradictory.

Unknown information that affects scope must block Change Planning or trigger Requirement Intent Gate.

---

## Module-boundary check

Before Change Planning, compare the proposed behavior with the current responsibility boundaries.

Use this structure:

```yaml
boundary_assessment:
  intended_change:
    primary_boundary: regex

  required_modules:
    - regex_classifier

  possible_crossings:
    - from: regex
      to: ai_model
      reason: uncertain cases may require model fallback
      status: requires_confirmation

  verdict:
    within_existing_design: false
    confirmation_required: true
```

Examples of material crossings:

- regex → AI model decision
- backend → frontend behavior
- extraction → schema definition
- local function → shared domain service
- synchronous path → queue
- in-process state → database/cache
- application config → cloud platform config

A crossing is not automatically wrong. It must be made explicit before execution.

---

## Persistence in memory.md

Persist architecture slices in the project `memory.md`.

Use an append-or-update structure:

```markdown
## Architecture Slice: <slice-id>

- Task:
- Generated:
- Verified against architecture.md:
- Status:

### Flow

```text
...
```

### Architecture pseudocode

```python
...
```

### Stable interfaces

- ...

### Runtime boundaries

- ...

### Confirmed / inferred / unknown

**Confirmed**
- ...

**Inferred**
- ...

**Unknown**
- ...

### Change-planning notes

- Safe boundary:
- Possible crossings:
- Invariants:

### Source references

- `path/to/file.py::Class.method`
- `tests/test_x.py::test_behavior`
```

Update an existing slice instead of adding duplicates when it describes the same path and is still relevant.

Do not store raw chain-of-thought. Store only architecture facts, evidence, decisions, and concise rationale.

---

## Consistency check with architecture.md

Before returning:

1. Compare the local slice with the relevant architecture section.
2. Record:
   - missing documentation,
   - stale documentation,
   - conflicting module ownership,
   - undocumented runtime boundaries,
   - outdated interface contracts.
3. Do not automatically rewrite `architecture.md` during analysis.

Return:

```yaml
architecture_sync:
  required_now: false
  required_after_implementation: true
  reason:
    - planned change adds a new OCR fallback branch
  target_sections:
    - Document Processing Flow
```

---

## Post-implementation architecture update

After implementation and verification, call the architecture-maintenance agent when any of these is true:

- module responsibility changed,
- a new branch was introduced,
- a stable interface changed,
- a runtime boundary was added or removed,
- state ownership changed,
- error propagation changed,
- a new external service or queue was introduced,
- `architecture.md` was already inconsistent.

The maintenance agent should receive:

```yaml
architecture_update_request:
  task_id: string
  original_slice_id: string
  verified_diff_summary: string
  changed_boundaries: []
  changed_interfaces: []
  changed_flow: []
  target_sections: []
```

Do not update `architecture.md` for purely internal implementation details that do not change architecture.

---

## Constraints

- Do not regenerate full project documentation.
- Analyze only the task-relevant architecture slice.
- Cite real modules, classes, functions, tests, and documentation sections.
- Do not modify source code.
- Do not perform Change Planning inside this skill.
- Do not silently resolve code/document conflicts.
- Do not expand implementation detail beyond what Change Planning needs.
- Do not represent inferred behavior as fact.
- Include async, queue, cache, transaction, and other runtime boundaries when relevant.
- Persist the result to `memory.md`.
- Request architecture maintenance after implementation when architecture changes.

---

## Completion checklist

- [ ] Invocation was necessary or skip reason was recorded.
- [ ] Task-relevant entrypoint and terminal output are identified.
- [ ] Data flow and major branches are represented.
- [ ] Stable interfaces are identified.
- [ ] Runtime boundaries are marked.
- [ ] Error propagation is represented.
- [ ] Confirmed, inferred, and unknown facts are separated.
- [ ] Module-boundary crossing is assessed.
- [ ] Consistency with architecture.md is checked.
- [ ] Architecture slice is persisted to memory.md.
- [ ] Post-implementation architecture sync requirement is recorded.
- [ ] Output is sufficient for Change Planning.
