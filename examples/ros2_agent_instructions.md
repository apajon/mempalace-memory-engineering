# Example: ROS 2 Agent Instructions

> **Scope.** An example instruction file for an AI coding agent working in a
> ROS 2 project that uses a governed memory layer (for example, MemPalace).
> It shows how to turn the wing/room/entry model into concrete operational
> rules: what memory to gather, in what order, what to persist, and what
> guardrails apply.
>
> This is a **reusable example**, not a fixed configuration. Placeholders like
> `<project_wing>` and `<project_name>` are meant to be replaced. The ROS 2
> specificity is intentional where it carries useful constraints (lifecycle,
> executors, callbacks); elsewhere the shape is generic.

For the underlying methodology, see
[docs/architecture.md](../docs/architecture.md),
[docs/retrieval.md](../docs/retrieval.md), and
[principles/rules.md](../principles/rules.md).

---

## How to Use This File

1. Copy this file into your project (e.g. as
   `.github/instructions/ros2-architecture-context.instructions.md` or the
   equivalent for your agent).
2. Replace every placeholder:
   - `<project_wing>` — your project's wing (e.g. `lifecore_ros2`, `myrobot`).
   - `<shared_wing>` — the shared platform wing (typically `ros2`).
   - `<project_name>` — the human-readable project name.
3. Keep or drop the ROS 2–specific rules depending on your stack. The
   context-gathering, persistence, and deduplication sections are
   stack-agnostic.

---

## 1. Context Gathering

Before modifying any component, query memory in this order:

1. **Project wing** (`<project_wing>`) — project-specific architecture rules,
   component contracts, local conventions, and project anti-patterns.
2. **Shared wing** (`<shared_wing>`) — general ROS 2 knowledge: lifecycle
   semantics, communication patterns, standard conventions.
3. **Merge results.** Project entries override shared entries **only when
   explicitly marked as a local override** (`label: local-override`).
4. **Fallback chain.** If the memory backend is unavailable, fall back to
   `docs/architecture.md`, then `README.md`, then workspace search. Memory
   unavailability must never block a task.

---

## 2. Retrieval Priority

```
[project wing]  →  [shared wing]  →  [local docs]  →  [code search]
       │
       ▼
  merged context
       │
       ▼
    decision
       │
       ▼
  persist (only if durable)
```

Within a wing/room, apply entries in type order:

1. `invariant` — applied unconditionally.
2. `decision` — scoped to the active wing and room.
3. `pattern` — matched to the current task.
4. `note` — supplementary context, used last.

Entries with `status: deprecated` are **not** surfaced unless explicitly
requested.

**Conflicts.** If a project entry and a shared entry contradict each other,
the project entry wins — but only when it is explicitly documented as a local
override. An undocumented contradiction is a potential inconsistency, not a
silent override. Flag it for human review.

---

## 3. Persistence

### 3.1 What to persist

Persist entries that are **durable**, **repeatable**, and **non-obvious**:

- Architecture decisions with rationale
- Inter-component contracts
- Confirmed anti-patterns (especially those that caused real bugs)
- Conventions that differ from ROS 2 defaults
- Rules that prevent known regressions

### 3.2 What not to persist

Do **not** persist:

- Debugging steps for a specific one-off issue
- Temporary session results
- Facts already present verbatim in docstrings or generated API docs
- One-time fixes with no recurring relevance
- Speculative ideas that have not been validated

The test: *if this knowledge were lost tomorrow, would we reconstruct it the
same way?* If yes, persist. If no, skip.

### 3.3 Before writing a new entry (deduplication)

Search the **target wing and room** for semantically similar content before
creating anything new. Scoping the comparison to the same wing and room keeps
the signal relevant.

Similarity guidance (not enforcement):

| Similarity   | Signal           | Action                                                                   |
|--------------|------------------|--------------------------------------------------------------------------|
| ≥ 0.86       | Near-duplicate   | Enrich the existing entry; do not create a new one.                      |
| 0.55 – 0.85  | Related content  | Review manually; create only if the new entry is genuinely distinct.     |
| < 0.55       | Likely distinct  | Creating a new entry is acceptable if persistence criteria are met.      |

**Type-aware exception.** If two entries are similar in content but differ in
type (e.g. `architecture-rule` vs. `anti-pattern`), keep them as separate
entries. Different types serve different purposes and must not be merged
automatically.

**When enriching.** Preserve the original rule. Extend or qualify it; do not
rewrite it from scratch. The original constraint must remain legible in the
updated entry.

If available, use a duplicate-check tool (e.g. `mempalace_check_duplicate`) as
a secondary guard.

### 3.4 Entry format

```
type: invariant | decision | pattern | note
label: architecture-rule | component-contract | anti-pattern | code-convention | migration-note
status: draft | active | under_review | deprecated
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD        # optional
version: N                     # optional

<one rule or contract, stated directly>

Rationale: <why this rule exists>

[see: <wing>/<room>] — <relationship to the referenced entry>   # optional
```

Rules:

- **One entry, one rule.** If you need two distinct headings, it is two
  entries.
- **Metadata is required.** `type`, `status`, and `created_at` must always be
  present. Missing metadata is a defect, not an omission.
- **Drafts need approval.** Agents may create entries with `status: draft`;
  they must not set `status: active` without human approval. This is
  non-negotiable for `invariant` and `decision` entries.

### 3.5 Scope assignment

- **Project wing** (`<project_wing>`): architecture decisions, component
  contracts, local conventions, project anti-patterns.
- **Shared wing** (`<shared_wing>`): general ROS 2 knowledge reusable across
  projects — lifecycle semantics, standard message patterns, tooling
  conventions.

If knowledge is reusable across projects, it belongs in the shared wing, not
the project wing. Do **not** copy shared content into project entries;
cross-reference it instead:

```
[see: <shared_wing>/lifecycle] — this component extends the state machine described there
```

---

## 4. ROS 2–Specific Rules (Keep, Adapt, or Drop)

The rules below are common ROS 2 constraints. They are shown here as worked
examples of what typically lives in a shared `ros2` wing. Keep the ones that
apply to your stack; drop the rest.

- Nodes must use `rclcpp::Node` (or `rclpy.node.Node`) as the base class.
- Callbacks (subscription, timer, service, action) must not block. Long-running
  work must be dispatched to a separate thread or an action server.
- Use a single-threaded executor unless a specific node has been explicitly
  documented as requiring a multi-threaded executor.
- All publishers and subscribers must be declared in the constructor
  (C++) or `__init__` (Python), not lazily inside callbacks.
- Lifecycle nodes must gate all message processing and publication on the
  node's current lifecycle state.

These are examples, not a mandatory baseline. Your shared wing is the source of
truth for your team.

---

## 5. Guardrails

1. **Read before write.** Do not write to memory without reading first.
2. **Atomic entries.** Keep entries atomic — one entry, one rule.
3. **Explicit types.** Tag the primary `type` (and optionally a `label`)
   whenever intent could be ambiguous.
4. **Explicit supersession.** Mark superseded entries as `status: deprecated`
   and reference the replacement. Silent deletion is not permitted.
5. **Maintenance is not optional.** Treat drift detection and supersession as
   part of the normal workflow, not as cleanup that can be deferred.
6. **Flag drift, don't paper over it.** If memory contradicts the current
   code, surface the discrepancy; do not silently use the code as truth or
   silently apply stale memory.
7. **Attribute decisions to memory.** When a behavior is driven by a memory
   entry, be able to name it: "following `no-blocking-callbacks` in
   `<shared_wing>/invariants`."

---

## 6. Worked Snippets

### 6.1 A shared-wing invariant

**Wing:** `<shared_wing>` · **Room:** `invariants`

```
type: invariant
label: architecture-rule
status: active
created_at: 2026-02-01
verified_at: 2026-04-10

ROS 2 callback functions (subscription, timer, service, action) must not block
for more than 1 ms. Long-running work must be dispatched to a separate thread
or implemented as an action server.

Rationale: blocking callbacks stall the executor and disrupt all other
callbacks sharing that executor context.
```

### 6.2 A project-wing local override

**Wing:** `<project_wing>` · **Room:** `architecture`

```
type: decision
label: local-override
status: active
created_at: 2026-03-05

This project uses a MultiThreadedExecutor for the navigation integration node,
overriding the shared default of a single-threaded executor.

Rationale: the navigation node coordinates two long-running action clients
whose callbacks would otherwise contend on a single executor thread.

[see: <shared_wing>/executors] — this entry is an explicit override of the
shared default.
```

Because this entry is labeled `local-override` and explicitly references the
shared rule it overrides, it resolves the conflict cleanly instead of creating
a silent contradiction.

---

*References:*
[docs/architecture.md](../docs/architecture.md) ·
[docs/retrieval.md](../docs/retrieval.md) ·
[docs/deduplication.md](../docs/deduplication.md) ·
[principles/rules.md](../principles/rules.md)
