# Architecture: Wings, Rooms, and Entries

This document describes a practical way to organize long-term memory in AI-assisted engineering workflows. It is not an official MemPalace workflow. It is a model that stays understandable as memory grows.

---

## 1. Purpose

As agents take on more work, they accumulate more context. If that context is not structured, the right rule becomes harder to retrieve at the right time.

The goal is simple: treat memory like an engineering artifact, not a pile of notes.

Each entry should have:

- a clear scope
- a clear subject
- a clear type
- a clear lifecycle status

---

## 2. Why flat memory fails

When memory is one undifferentiated list:

- stale and current guidance look the same
- shared knowledge bleeds into project rules
- the same rule shows up in several phrasings
- useful signal gets buried in noise
- context space is wasted

Structured memory fixes this by giving every entry a clear place and role.

---

## 3. Design goals

| Goal | In practice |
|------|-------------|
| **Scoped** | Project and shared knowledge stay separate |
| **Retrievable** | Agents gather context in a predictable order |
| **Deduplicated** | The same rule does not appear in competing forms |
| **Durable** | Important knowledge survives sessions and tool changes |
| **Graceful** | Work continues if memory is unavailable |

These outcomes require disciplined writing and retrieval, not specialized tooling.

---

## 4. Core model

### Wings

A wing is the top-level scope boundary.

Examples:

- a specific project, like `lifecore_ros2`
- shared technical knowledge, like `ros2` or `react`

Agents load the active wing by default. Cross-wing retrieval should be explicit.

**Naming guidance:**

- use lowercase names
- use underscores for multi-word identifiers
- keep names short and domain-specific
- treat a wing as a scope boundary, not a tag

### Rooms

A room is a subject area inside a wing.

**Examples inside `ros2`:**

- `ros2/lifecycle`
- `ros2/nodes`
- `ros2/anti-patterns`

**Standard room slugs** when content justifies them:

| Room | Purpose |
|------|---------|
| `architecture` | High-level design decisions and boundaries |
| `components` | Component interfaces and extension points |
| `communication` | APIs, message flow, and protocols |
| `configuration` | Parameters, environment, and config behavior |
| `contracts` | Agreements between components |
| `anti-patterns` | Confirmed mistakes to avoid |
| `conventions` | Naming, style, and tooling conventions |
| `validation` | Testing rules and CI expectations |
| `migration-notes` | Breaking changes and upgrade guidance |
| `incident-log` | Specific incidents |
| `debugging` | Repeatable investigation workflows |
| `observability` | Logs, metrics, traces, and runtime visibility |
| `failure-modes` | Recurring failure classes and recovery strategies |

Start with `architecture` and `anti-patterns`. Add more rooms only when real content justifies them.

### Entries

An entry is the atomic unit of durable memory.

Each entry should include:

- **Type**
- **Content**
- **`created_at`**
- **`status`**
- **`verified_at`** when relevant

One entry should contain one durable rule or fact.

### Entry types

| Type | Description | Weight |
|------|-------------|--------|
| `invariant` | A rule that must not be violated | Highest |
| `decision` | A deliberate design or architecture choice | High |
| `pattern` | A reusable approach to a class of problem | Medium |
| `note` | Supplementary context | Low |

`deprecated` is not a type. It is a **status**.

Optional labels can add precision for humans:

```text
[label: architecture-rule]
[label: component-contract]
[label: anti-pattern]
[label: code-convention]
[label: reusable-pattern]
[label: migration-note]
```

Labels do not replace the primary type.

### Hierarchy summary

```text
wing/
  room/
    entry (type: invariant | decision | pattern | note, status: draft | active | under_review | deprecated)
```

Every entry should have a stable address such as `wing/room/entry-id`.

---

## 5. Scope separation

The main design decision is where to draw scope boundaries.

**Rule: separate by scope, not by workspace layout.**

Many teams end up with two main wings:

| Wing | Scope | Typical contents |
|------|-------|------------------|
| `<project_name>` | Project-specific | Local decisions, contracts, overrides, anti-patterns |
| `<technology>` or `shared` | Reusable knowledge | Framework or platform knowledge used across projects |

If knowledge is reusable across projects, it belongs in the shared wing.

---

## 6. Retrieval model

Agents should gather context in this order:

```text
1. Project wing
2. Shared wing
3. Local docs
4. Code search
```

Within retrieved memory, apply:

1. `invariant`
2. `decision`
3. `pattern`
4. `note`

Entries with `status: deprecated` stay out of default retrieval.

---

## 7. Conflict handling

If a project entry and a shared entry cover the same topic, the project entry only wins when it is explicitly documented as a local override.

Without that marker, the conflict should be treated as a review issue, not resolved silently.

**Example:**

```text
type: decision
label: local-override
status: active

This project does not follow the shared `ros2/lifecycle` activation sequence
because the hardware driver requires a custom initialization step before
activate() can succeed.
```

---

## 8. Persistence rules

### What to persist

Persist entries that are:

- durable
- repeatable
- non-obvious

Good examples include architecture decisions, component contracts, anti-patterns tied to real failures, and conventions that differ from defaults.

### What not to persist

Do not persist:

- one-off debugging notes
- temporary session context
- information already captured verbatim in code docs
- speculative rules that are not yet validated

### Deduplication

Before adding a new entry, search the target wing and room first.

- if an entry already covers the rule, enrich it
- if the new entry replaces an old one, mark the old one `deprecated`
- do not create duplicates with slightly different wording

Similarity thresholds can help:

| Similarity | Signal | Suggested action |
|-----------|--------|-----------------|
| >= 0.86 | Near-duplicate | Enrich the existing entry |
| 0.55 - 0.85 | Possibly overlapping | Review manually |
| < 0.55 | Likely distinct | A new entry may be justified |

Type still matters. Similar content can still belong in separate entries if the types differ.

### Freshness metadata

For entries that may drift:

```text
status: active
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD   (optional)
version: N                (optional)
```

Use `status: under_review` when the surrounding system has changed and the entry has not been re-checked.

### Cross-references

If a project entry depends on a shared rule, reference it instead of copying it:

```text
[see: ros2/lifecycle] — this component extends the state machine described there
```

---

## 9. Graceful degradation

Memory should help work continue, not block it.

If the memory backend is unavailable:

```text
memory backend -> in-repo architecture docs -> README -> workspace search
```

---

## 10. Practical starting point

You do not need a large memory system on day one.

A simple starting point is often enough:

- one project wing
- one shared wing
- two rooms: `architecture` and `anti-patterns`

Add structure only when real usage justifies it.

---

## 11. Relationship to other docs

| Document | What it covers |
|----------|----------------|
| [retrieval.md](retrieval.md) | Retrieval ordering, scoping, filtering, and context pressure |
| [persistence.md](persistence.md) | Durability expectations and storage requirements |
| [deduplication.md](deduplication.md) | How to prevent duplicate or drifting entries |