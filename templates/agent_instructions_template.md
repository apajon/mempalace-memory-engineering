# Template: Agent Instructions

A reusable template for writing instruction files that govern how an AI coding
agent uses structured memory in a project. Copy this file, fill in the
placeholders, and keep the sections that apply.

Replace every `<placeholder>`. Sections marked *(optional)* may be removed
when they do not apply. Do **not** remove the persistence, deduplication, or
guardrail sections — they are the minimum viable contract with memory.

Related reading:
[docs/architecture.md](../docs/architecture.md) ·
[docs/retrieval.md](../docs/retrieval.md) ·
[docs/deduplication.md](../docs/deduplication.md) ·
[principles/rules.md](../principles/rules.md).

---

```markdown
# <project_name> — Agent Instructions

> **Scope.** These instructions govern how AI coding agents gather context,
> reason, and persist knowledge when working on <project_name>. They apply to
> code generation, review, debugging, and documentation tasks.

---

## 1. Wings and Rooms

This project uses the following memory layout:

- **Project wing:** `<project_wing>`
  - `<project_wing>/architecture`   — project-specific design rules
  - `<project_wing>/anti-patterns`  — confirmed local mistakes
  - `<project_wing>/conventions`    — naming, style, tooling specific to this project
  - `<project_wing>/<room>`         — (optional) additional rooms as content justifies

- **Shared wing:** `<shared_wing>`
  - `<shared_wing>/<room>`          — reusable platform or framework knowledge

Add operational rooms only once real content justifies them:
`incident-log`, `debugging`, `observability`, `failure-modes`. Keep their
boundaries explicit — see
[docs/architecture.md § Rooms](../docs/architecture.md#rooms).

---

## 2. Context Gathering

Before producing output or modifying code, query memory in this order:

1. **`<project_wing>`** — project-specific rules, contracts, and anti-patterns.
2. **`<shared_wing>`** — general platform / framework knowledge.
3. **Local docs** — `docs/`, `README.md`, in-repo architecture files.
4. **Code search** — grep, symbol lookup, file search.

Within a wing/room, apply entries in type order:

1. `invariant` — applied unconditionally.
2. `decision` — scoped to the active wing and room.
3. `pattern` — matched to the current task.
4. `note` — supplementary context, used last.

Entries with `status: deprecated` are **not** surfaced unless explicitly
requested.

---

## 3. Retrieval and Conflict Policy

- Retrieval is **scoped to the active wing** by default. Cross-wing retrieval
  requires explicit instruction.
- Project entries override shared entries **only when explicitly marked as a
  local override** (`label: local-override`). An undocumented contradiction
  between a project entry and a shared entry is a potential inconsistency:
  flag it for human review rather than resolving it silently.
- If no relevant memory exists, say so. A missing entry may mean no constraint
  has been written yet, not that no constraint exists.

---

## 4. Fallback Behavior (Graceful Degradation)

Memory unavailability must never block a task. If the memory backend is not
reachable, fall back in this order:

```
memory backend → <fallback_doc_1, e.g. docs/architecture.md>
              → <fallback_doc_2, e.g. README.md>
              → workspace search (grep / symbol lookup)
```

Do not halt, do not error out, do not ask the user to restart the backend.
Continue with the best available sources and surface that memory was
unavailable in the session summary.

---

## 5. Persistence Criteria

Persist entries that are **durable**, **repeatable**, and **non-obvious**:

- Architecture decisions with rationale
- Inter-component contracts
- Confirmed anti-patterns (especially those tied to real incidents)
- Conventions that differ from tool or framework defaults
- Rules that prevent known regressions

Do **not** persist:

- Debugging steps for a specific one-off issue
- Temporary session results
- Facts already in docstrings or generated API docs
- One-time fixes without recurring relevance
- Speculative ideas that have not been validated

**Test:** *if this knowledge were lost tomorrow, would we reconstruct it the
same way?* If yes, persist. If no, skip.

---

## 6. Deduplication Rules

Before writing a new entry, search the **target wing and room** for
semantically similar content. Scoping the comparison is what keeps the signal
useful.

| Similarity   | Signal           | Action                                                               |
|--------------|------------------|----------------------------------------------------------------------|
| ≥ 0.86       | Near-duplicate   | Enrich the existing entry; do not create a new one.                  |
| 0.55 – 0.85  | Related content  | Review manually; create only if genuinely distinct.                  |
| < 0.55       | Likely distinct  | Creating a new entry is acceptable if persistence criteria are met.  |

**Type-aware exception.** Entries that are similar in content but differ in
type (e.g. `architecture-rule` vs. `anti-pattern`) stay separate. Different
types serve different purposes and must not be merged automatically.

**When enriching.** Preserve the original rule and its rationale. Extend or
qualify it; do not rewrite from scratch.

See [docs/deduplication.md](../docs/deduplication.md) for the full policy.

---

## 7. Conflict Policy

- **Invariants win over convenience.** An agent must not violate an invariant
  for the sake of brevity, speed, or user preference. Surface the conflict
  explicitly and wait for human resolution.
- **Decisions are stable until revised.** Do not re-litigate an active
  `decision` in every session. Propose alternatives only when explicitly
  asked to reconsider.
- **Drift is flagged, not silently reconciled.** If memory contradicts the
  current code, report the discrepancy. Do not treat the current code as
  truth, and do not silently apply stale memory.
- **Supersession is explicit.** Replaced entries are marked
  `status: deprecated` and reference the replacement. Silent deletion is not
  permitted.

---

## 8. Authoring Rules

- Agents may **draft** new memory entries. Agents must **not** set
  `status: active` without human approval. This is non-negotiable for
  `invariant` and `decision` entries.
- Drafted entries must include `type`, `status: draft`, and `created_at`.
  Missing metadata is a defect.
- Keep entries **atomic**: one entry, one rule.
- Be **specific**: no vague entries. If the content cannot be made specific,
  flag the ambiguity for human resolution.
- When a memory entry drives a decision in the current session, be prepared
  to **name the entry** that drove it.

---

## 9. Task-Specific Behavior

> *Customize per task type. Keep each section short and reference specific
> rooms or entries rather than abstract principles.*

### 9.1 Code Generation

When generating code in this project:

- [Specific rule derived from `<project_wing>/architecture`]
- [Specific rule derived from `<shared_wing>`]
- [Convention from `<project_wing>/conventions`]

### 9.2 Code Review

When reviewing code:

1. Load `<project_wing>/architecture` and any `<project_wing>/anti-patterns`
   relevant to the change.
2. Any invariant violation is a **blocking** issue, not a suggestion.
3. Conflicts with active decisions must be flagged; they may be resolved by
   discussion, but not silently ignored.

### 9.3 Debugging

When debugging a reported issue:

1. Identify the affected component and interface before proposing a fix.
2. Check for matching entries in `<project_wing>/failure-modes`,
   `<project_wing>/incident-log`, or `<project_wing>/anti-patterns`.
3. If the fix implies a design change, determine whether a new `decision` or
   `invariant` entry is warranted, and draft it for review.

### 9.4 Memory Updates *(optional)*

When a session reveals an inaccurate or outdated entry:

1. Note the discrepancy explicitly in the session output.
2. Draft an updated entry (`status: draft`) for human review.
3. Do **not** rely on the inaccurate entry for further decisions in this
   session.

---

## 10. Project Scope

The agent's scope in this project is: `<define_scope>`
(for example: "backend service code only", "ROS 2 nodes in the
`<package_name>` package", "database schema and migration files").

Do not make changes or recommendations outside this scope without explicit
instruction.
```

---

## Notes on Using This Template

- **Be specific.** Generic instructions ("write good code") are not
  actionable. Reference concrete rooms, entries, or invariants.
- **Keep it stable.** This file is loaded at the start of every session.
  Avoid embedding session-specific context in it; session context belongs in
  drafted notes.
- **Separate *must* from *may*.** "Must" maps to invariants. "May" maps to
  patterns and notes. Agents treat the two differently during retrieval.
- **Do not remove the memory-hygiene sections.** Context gathering, fallback,
  persistence, deduplication, conflict policy, and authoring rules together
  form the minimum contract. Removing one breaks the others.
