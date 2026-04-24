# Architecture: Wings, Rooms, and Entries

This document describes a methodology for organizing long-term memory in AI-assisted engineering workflows. It is not an official MemPalace workflow or a mandatory standard — it is a structured approach that has proven practical when agents need to share persistent context across sessions and projects.

---

## 1. Purpose

As AI-assisted engineering grows more complex, agents accumulate more context. Without deliberate structure, that context degrades. This document describes how to design memory so that agents consistently apply the right rules, without being re-explained every session.

The core idea: treat memory as an engineering artifact, not a pile of notes. Every entry gets a known location, a type, and a weight.

---

## 2. Problem: Why Flat Memory Fails

When all entries exist at the same level, without hierarchy or taxonomy, several problems emerge as volume grows:

- **Agents retrieve outdated entries alongside current ones.** A stale decision and an active one look identical during retrieval.
- **Project-specific rules bleed into general knowledge.** Platform conventions get mixed with local overrides, and neither is trustworthy.
- **The same fact accumulates in multiple phrasings.** Retrieval surfaces all of them. They don't always agree.
- **Useful insights vanish into noise.** A well-written invariant disappears next to fifty session notes that should never have been stored.
- **Context space is wasted.** Retrieved entries that don't belong crowd out the ones that do.

Structured memory solves this by assigning every entry a location, a type, and a scope.

---

## 3. Design Goals

A well-governed memory layer should satisfy these properties:

| Goal | What it means in practice |
|------|--------------------------|
| **Scoped** | Project knowledge and shared knowledge are separated. Retrieval stays within scope by default. |
| **Retrievable** | Agents can find the right context in a predictable order. |
| **Deduplicated** | The same rule does not exist in two places with different phrasings. |
| **Durable** | Decisions persist beyond session boundaries and tool restarts. |
| **Graceful** | Agents do not break when memory is unavailable. |

None of these require specialized tooling. They require discipline in how you write to and read from your memory store.

---

## 4. Core Model

### Wings

A wing is the top-level namespace. It defines a domain of concern and a scoping boundary.

An agent working within a wing loads memory from that wing by default. Cross-wing retrieval is explicit and deliberate — not automatic.

**Examples:**
- `lifecore_ros2` — a specific robotics project
- `ros2` — general ROS 2 knowledge, reusable across projects
- `myapp` — a web application project
- `react` — shared React conventions

**Naming conventions:**
- Lowercase identifiers, underscores for multi-word names
- Short and domain-specific
- A wing is not a tag. It is a boundary.

### Rooms

A room is a category within a wing. Rooms make retrieval predictable and keep entries searchable by subject.

**Examples within a `ros2` wing:**
- `ros2/lifecycle` — lifecycle state machine semantics
- `ros2/nodes` — per-node constraints and patterns
- `ros2/anti-patterns` — confirmed mistakes to avoid

**Standard room slugs** (create only when content justifies them):

| Room | Purpose |
|------|---------|
| `architecture` | High-level design decisions, layer definitions |
| `components` | Component contracts, interfaces, extension points |
| `communication` | Message passing, API patterns, protocols |
| `configuration` | Config files, parameters, environment setup |
| `contracts` | Inter-component agreements, API guarantees |
| `anti-patterns` | Confirmed mistakes to avoid |
| `conventions` | Naming, style, tooling conventions |
| `validation` | Testing rules, CI gates, quality checks |
| `migration-notes` | Breaking changes, version upgrades, deprecation paths |
| `incident-log` | Specific incidents: symptoms, diagnosis, root cause, fix, prevention |
| `debugging` | Diagnostic commands, debug patterns, investigation workflows |
| `observability` | Logging, metrics, tracing, runtime visibility expectations |
| `failure-modes` | Known failure classes: triggers, symptoms, recovery strategy |

Start with `architecture` and `anti-patterns`. Add rooms only when content justifies them. Do not preallocate empty rooms.

Keep room boundaries explicit: `incident-log` records a specific event that happened; `failure-modes` generalizes recurring classes of failure; `debugging` captures how to investigate; `observability` captures how to make investigation possible earlier.

### Entries

An entry is a single memory item within a room. It is the atomic unit of memory.

Every entry should have:
- **Type** — what kind of information it represents (see below)
- **Content** — the actual information, written in clear, direct language
- **`created_at`** — when the entry was first written
- **`status`** — `draft`, `active`, `under_review`, or `deprecated`
- **`verified_at`** (optional) — when the entry was last confirmed accurate

One entry, one rule. If an entry requires two distinct headings to organize its content, it is likely two entries.

### Entry Types

| Type | Description | Typical Weight |
|------|-------------|----------------|
| `invariant` | A rule or constraint that must never be violated | Highest |
| `decision` | An architectural or design choice with rationale | High |
| `pattern` | A recurring solution or approach to a class of problem | Medium |
| `note` | Contextual or supplementary information | Low |
| `deprecated` | Superseded content retained for traceability | Not surfaced by default |

Type determines how an agent treats the entry during retrieval. An agent must not override an `invariant` without explicit human instruction. A `note` may be deprioritized when context space is constrained.

**Finer-grained labels.** The five types above are the primary taxonomy. Within an entry's content, a secondary label may clarify intent when useful:

```
[label: architecture-rule]    # usually an invariant or decision
[label: component-contract]   # usually a decision
[label: anti-pattern]         # usually an invariant ("must not")
[label: code-convention]      # usually a pattern
[label: reusable-pattern]     # usually a pattern
[label: migration-note]       # usually a note or decision
```

Labels are optional and do not replace the primary type. Retrieval and weighting are driven by the primary type; labels exist for human clarity.

### Hierarchy Summary

```
wing/
  room/
    entry (type: invariant | decision | pattern | note | deprecated)
```

Every entry has a complete address: `wing/room/entry-id`. This address is stable and referenceable across sessions.

---

## 5. Scope Separation

The most important structural decision is how to split scope between wings.

**Rule: separate by scope, not by workspace.**

In practice, most setups land on two wings:

| Wing | Scope | What goes here |
|------|-------|----------------|
| `<project_name>` | Project-specific | Architecture decisions, component contracts, local conventions, project-specific anti-patterns |
| `<technology>` or `shared` | Shared / transverse | General platform or framework knowledge — reusable across projects |

**Example:** a team building `lifecore_ros2` might use a `lifecore_ros2` wing for project-specific rules and a `ros2` wing for general ROS 2 knowledge. A web team might use `myapp` and `react`.

**The rule:** if knowledge is reusable across projects, it belongs in the shared wing, not the project wing.

Violating this rule creates the "Project Island" problem: shared knowledge gets duplicated across project wings, diverges over time, and must be maintained in multiple places. When you find yourself copying a rule from one project wing into another, that is the signal to move it to the shared wing.

---

## 6. Retrieval Model

When an agent gathers context before making a decision or change, it should query memory in this order:

```
1. Project wing  →  project-specific rules take precedence
2. Shared wing   →  shared knowledge fills in what the project wing lacks
3. Local docs    →  README, architecture files, inline comments
4. Code search   →  grep / symbol search as last resort
```

Entries from both wings are merged into a single context. The agent then reasons from that merged context.

**Within-type ordering** (after scope is resolved):

1. `invariant` — applied unconditionally
2. `decision` — scoped to the active wing and room
3. `pattern` — matched to the current task
4. `note` — supplementary context, used last

Entries marked `deprecated` are not surfaced unless explicitly requested.

**Practical pattern for instruction files:**

```markdown
## Context Gathering

1. Query the project wing for project-specific rules.
2. Query the shared wing for general knowledge.
3. Merge results. Project rules override shared rules only when explicitly marked.
4. If the memory backend is unavailable, fall back to in-repo architecture docs, then README, then workspace search.
```

---

## 7. Conflict Handling

When a project wing entry and a shared wing entry address the same topic, the project entry takes precedence — but only when it is explicitly documented as a local override.

A project entry that simply contradicts a shared rule without explanation should be treated as a potential inconsistency, not a silent override. Agents should flag this for human review rather than resolving it independently.

**Example of an explicit override:**

```
type: decision
label: local-override
status: active

This project does not follow the shared `ros2/lifecycle` activation sequence
because the hardware driver requires a custom initialization step before
activate() can succeed. See ticket LIFECORE-42 for context.
```

Without a statement like this, conflicting entries are a warning sign, not a precedence rule.

---

## 8. Persistence Rules

### What to persist

Store entries that are:
- **Durable** — will remain relevant beyond the current session
- **Repeatable** — you will want this rule enforced again in the future
- **Non-obvious** — not trivially inferred from reading the code

Good candidates: architecture decisions with rationale, inter-component contracts, confirmed anti-patterns, conventions that differ from tool defaults, rules that prevent known regressions.

### What not to persist

Do not store: debugging steps for a specific one-off issue, temporary session results, information already present verbatim in docstrings, one-time fixes with no recurring relevance, speculative ideas not yet validated.

The test: *if this knowledge were lost tomorrow, would we reconstruct it and write it the same way?* If yes, persist. If no, skip.

### Deduplication

Before writing a new entry, search the target wing and room for semantically similar content.

- If an existing entry covers the same information: **enrich** the existing entry rather than creating a new one.
- If the new entry supersedes an older one: mark the old entry `status: deprecated` with a reference to its replacement.
- Never create two entries with the same core rule in different phrasings.
- When enriching an existing entry, preserve its original rule and extend it — do not rewrite from scratch.

Teams that want a more structured signal can use similarity scores as guidance (not enforcement). Scoped comparison within the target wing and room keeps the signal relevant:

| Similarity | Signal | Suggested action |
|-----------|--------|-----------------|
| ≥ 0.86 | Near-duplicate | Enrich the existing entry; do not create a new one |
| 0.55 – 0.85 | Related content | Review manually; create only if the new entry captures a genuinely distinct rule |
| < 0.55 | Likely distinct | Creating a new entry is acceptable if persistence criteria are met |

**Type-aware exception:** two entries that are similar in content but differ in type (e.g., `architecture-rule` vs. `anti-pattern`) should remain separate. Different types serve different purposes.

These thresholds are a refinement on top of disciplined writing — not a replacement for human judgment. See [deduplication.md](deduplication.md) for a compact reference.

### Freshness metadata

For entries that may evolve, include a metadata header:

```
status: active
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD   (optional)
version: N                (optional)
```

Set `status: under_review` when the surrounding codebase has changed significantly but the entry has not been re-verified. Stale memory is worse than no memory.

### Cross-references

When a project entry depends on shared knowledge, reference it rather than copying it:

```
[see: ros2/lifecycle] — this component extends the native state machine described there
```

This keeps project entries lean and makes the dependency explicit.

---

## 9. Graceful Degradation

Memory unavailability must never block a task.

When memory tools are not accessible, agents must continue with local sources — not halt, not error, not ask for intervention. Design your instruction files so the fallback chain is explicit:

```
memory backend → in-repo architecture docs → README → workspace search
```

An agent that refuses to work because its memory backend is down has turned memory into a single point of failure. That is a design defect, not a safety measure.

---

## 10. Practical Starting Point

You do not need to implement this fully on day one.

A minimal setup that covers the highest-value cases:
- **1 project wing** — for the current project
- **1 shared wing** — for reusable platform or framework knowledge
- **2 rooms** — `architecture` and `anti-patterns`

Add rooms only when content justifies them. Refine the scope split only when you notice mis-scoping patterns. The structure is a tool, not a goal.

**Example: a ROS 2 project**

A team building `lifecore_ros2` sets up:

- Wing `lifecore_ros2`, room `architecture`:
  ```
  type: invariant
  label: architecture-rule
  status: active
  created_at: 2025-03-01

  Topic components must gate all message processing and publication on the node's
  activation state. Subscribers must check node state before forwarding messages.
  Publishers must only publish when the node is in the ACTIVE state.

  Rationale: prevents message processing during partial initialization or cleanup.

  [see: ros2/lifecycle] for the state machine this rule extends.
  ```

- Wing `ros2`, room `lifecycle`:
  ```
  type: decision
  label: shared-knowledge
  status: active
  created_at: 2025-01-15

  ROS 2 lifecycle nodes follow a standard state machine: Unconfigured → Inactive →
  Active → Finalized. Transitions are triggered by configure(), activate(), deactivate(),
  cleanup(), shutdown(). Do not introduce internal states that shadow or diverge from
  this machine.
  ```

The project rule extends the shared rule. The shared rule stays in the shared wing. No copying.

A Copilot instruction file then says:

> Before modifying any component lifecycle code, query `lifecore_ros2` then `ros2`. Merge results. Project rules take precedence only when marked as local overrides.

That is the full loop. No manual re-explanation required per session.

---

## 11. Relationship to Other Docs

| Document | What it covers |
|----------|---------------|
| [retrieval.md](retrieval.md) | Detailed retrieval ordering, scoping, filtering, and context window handling |
| [persistence.md](persistence.md) | Durability expectations, storage backend requirements, human readability |
| [deduplication.md](deduplication.md) | Compact reference for the deduplication policy |
| [anti_patterns.md](anti_patterns.md) | Memory design failures and how to fix them |
| [maintenance.md](maintenance.md) | Keeping memory healthy over time: drift, review cycles, supersession |
| [principles/rules.md](../principles/rules.md) | Behavioral rules for agents using memory |
| [principles/invariants.md](../principles/invariants.md) | Invariant scope and governance |
| [templates/](../templates/) | Entry and agent instruction templates |
| [examples/](../examples/) | Concrete worked examples |
