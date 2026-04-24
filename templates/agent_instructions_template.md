# Template: Agent Instructions

This template helps you write instruction files for AI coding agents that rely on structured memory.

Copy the sections you need, replace every placeholder, and remove any optional section that does not apply. Keep the memory hygiene sections. They are the minimum contract that keeps agent behavior predictable.

Related reading:
[docs/architecture.md](../docs/architecture.md) ·
[docs/retrieval.md](../docs/retrieval.md) ·
[docs/deduplication.md](../docs/deduplication.md) ·
[principles/rules.md](../principles/rules.md)

---

```markdown
# <project_name> — Agent Instructions

> **Scope.** These instructions define how AI coding agents gather context,
> reason, and persist knowledge when working on <project_name>.

---

## 1. Wings and Rooms

- **Project wing:** `<project_wing>`
  - `<project_wing>/architecture`
  - `<project_wing>/anti-patterns`
  - `<project_wing>/conventions`
  - `<project_wing>/<room>` when justified

- **Shared wing:** `<shared_wing>`
  - `<shared_wing>/architecture`
  - `<shared_wing>/conventions`
  - `<shared_wing>/<room>` when justified

Operational rooms such as `incident-log`, `debugging`, `observability`, and `failure-modes` should be added only when they carry real recurring content.

---

## 2. Context Gathering

Gather context in this order:

1. `<project_wing>`
2. `<shared_wing>`
3. local docs
4. code search

Within any room, apply:

1. `invariant`
2. `decision`
3. `pattern`
4. `note`

Entries with `status: deprecated` stay out of default retrieval.

---

## 3. Retrieval and conflict policy

- retrieval is scoped to the active wing by default
- cross-wing retrieval must be explicit
- project entries only override shared entries when marked `label: local-override`
- undocumented contradictions should be flagged for human review
- missing memory is not proof that no constraint exists

---

## 4. Fallback behavior

Memory unavailability must not block a task.

```text
memory backend -> <fallback_doc_1> -> <fallback_doc_2> -> workspace search
```

Do not halt the task because the backend is unavailable.

---

## 5. Persistence criteria

Persist entries that are durable, repeatable, and non-obvious.

Good candidates:

- architecture decisions with rationale
- inter-component contracts
- confirmed anti-patterns tied to real failures
- conventions that differ from framework defaults
- rules that prevent known regressions

Do not persist:

- one-off debugging steps
- temporary session results
- facts already present verbatim in docs or docstrings
- one-time fixes with no recurring relevance
- speculative ideas that are not yet validated

---

## 6. Deduplication rules

Before writing a new entry, search the target wing and room first.

| Similarity | Signal | Action |
|------------|--------|--------|
| >= 0.86 | Near-duplicate | Enrich the existing entry |
| 0.55 - 0.85 | Related content | Review manually |
| < 0.55 | Likely distinct | A new entry may be acceptable |

Different types can still coexist when they serve different retrieval roles.

---

## 7. Conflict policy

- invariants are binding
- decisions remain stable until explicitly revised
- if memory and code disagree, flag the discrepancy
- supersession must be explicit

---

## 8. Authoring rules

- agents may draft new entries
- agents must not set `status: active` without human approval
- drafts must include `type`, `status: draft`, and `created_at`
- keep entries atomic
- prefer specific statements over vague guidance
- if memory drives a decision, be prepared to name the entry

---

## 9. Task-specific behavior

### 9.1 Code generation

- [Specific rule derived from `<project_wing>/architecture`]
- [Specific rule derived from `<shared_wing>/architecture` or `<shared_wing>/conventions`]
- [Specific project convention from `<project_wing>/conventions`]

### 9.2 Code review

1. Load relevant architecture and anti-pattern entries first.
2. Treat invariant violations as blocking issues.
3. Flag conflicts with active decisions.

### 9.3 Debugging

1. Identify the affected component and interface first.
2. Check `failure-modes`, `incident-log`, and `anti-patterns` if those rooms exist.
3. If the fix implies a lasting design change, draft a memory update for review.

### 9.4 Memory updates *(optional)*

1. note the discrepancy clearly
2. draft an updated entry for review
3. avoid relying on the inaccurate entry for the rest of the session

---

## 10. Project scope

The agent's scope in this project is: `<define_scope>`

Do not make changes or recommendations outside that scope without explicit instruction.
```

---

## Notes on using this template

- be specific
- keep the file stable
- separate what the agent **must** do from what it **may** do
- keep the sections on context gathering, fallback behavior, persistence, deduplication, conflict handling, and authoring