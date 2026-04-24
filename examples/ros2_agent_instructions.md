# Example: ROS 2 Agent Instructions

This example shows how a ROS 2 project can turn the wing/room/entry model into practical agent instructions.

Replace the placeholders with project-specific values and keep only the ROS 2 rules that actually apply to your stack.

---

## How to use this file

1. Copy it into your project.
2. Replace every placeholder.
3. Keep the memory hygiene sections even if you trim the stack-specific rules.

---

## 1. Context gathering

Before modifying a component, gather memory in this order:

1. **Project wing** (`<project_wing>`)
2. **Shared wing** (`<shared_wing>`)
3. **Merge results** — project entries only override shared entries when they are explicit local overrides
4. **Fallback chain** — `docs/architecture.md`, then `README.md`, then workspace search if the backend is unavailable

Memory unavailability should never block a task.

---

## 2. Retrieval priority

```text
[project wing] -> [shared wing] -> [local docs] -> [code search]
        |
        v
   merged context
        |
        v
   act or reason
        |
        v
 persist only if durable
```

Within a room, apply:

1. `invariant`
2. `decision`
3. `pattern`
4. `note`

Entries with `status: deprecated` stay out of default retrieval.

---

## 3. Persistence

### 3.1 What to persist

Persist entries that are durable, repeatable, and non-obvious.

Good examples:

- architecture decisions with rationale
- inter-component contracts
- confirmed anti-patterns tied to real bugs
- conventions that differ from ROS 2 defaults
- rules that prevent known regressions

### 3.2 What not to persist

Do not persist:

- one-off debugging steps
- temporary session results
- facts already present verbatim in docs or docstrings
- one-time fixes with no likely recurrence
- speculative ideas that are not yet validated

### 3.3 Before writing a new entry

Search the target wing and room first.

| Similarity | Signal | Action |
|------------|--------|--------|
| >= 0.86 | Near-duplicate | Enrich the existing entry |
| 0.55 - 0.85 | Related | Review manually |
| < 0.55 | Likely distinct | A new entry may be valid |

Different types can still coexist if they serve different retrieval purposes.

### 3.4 Entry format

```text
type: invariant | decision | pattern | note
label: architecture-rule | component-contract | anti-pattern | code-convention | migration-note
status: draft | active | under_review | deprecated
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD        # optional
version: N                     # optional

<one durable rule or fact>

Rationale: <why it matters>
```

Rules:

- one entry, one durable rule
- required metadata must be present
- agents may draft entries, but must not activate them without human approval

### 3.5 Scope assignment

- **Project wing** (`<project_wing>`) — local architecture, contracts, conventions, anti-patterns
- **Shared wing** (`<shared_wing>`) — reusable ROS 2 knowledge used across projects

If knowledge is reusable across projects, store it in the shared wing and cross-reference it from project memory instead of copying it.

---

## 4. ROS 2-specific rules

Common examples for a shared `ros2` wing:

- Nodes should use `rclcpp::Node` or `rclpy.node.Node` as the base class.
- Callbacks should not block.
- Use a single-threaded executor unless a node has a documented reason not to.
- Publishers and subscribers should be declared in the constructor or `__init__`.
- Lifecycle nodes should gate message handling and publication on lifecycle state.

These are examples, not a mandatory baseline.

---

## 5. Guardrails

1. Read before write.
2. Keep entries atomic.
3. Use explicit types and status values.
4. Mark supersession explicitly.
5. Treat maintenance as part of the workflow.
6. If memory and code disagree, surface the discrepancy.
7. If memory drives a decision, be able to name the entry that drove it.

---

## 6. Worked snippets

### 6.1 A shared-wing invariant

**Wing:** `<shared_wing>` · **Room:** `anti-patterns`

```text
type: invariant
label: anti-pattern
status: active
created_at: 2026-02-01
verified_at: 2026-04-10

Do not block ROS 2 callback functions for more than 1 ms. Long-running work
must be dispatched to a separate thread or implemented as an action server.

Rationale: blocking callbacks stalls the executor and delays unrelated work.
```

### 6.2 A project-wing local override

**Wing:** `<project_wing>` · **Room:** `architecture`

```text
type: decision
label: local-override
status: active
created_at: 2026-03-05

This project uses a MultiThreadedExecutor for the navigation integration node,
overriding the shared default of a single-threaded executor.

Rationale: the navigation node coordinates two long-running action clients
whose callbacks would otherwise contend on a single executor thread.

[see: <shared_wing>/architecture/single-threaded-executor-default]
```

Because this entry is an explicit local override, it resolves the conflict cleanly.