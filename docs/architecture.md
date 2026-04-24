# Architecture: Wings, Rooms, and Entries

This document describes the structural model used to organize long-term memory in AI-assisted engineering workflows.

---

## The Problem with Flat Memory

When memory entries are stored without hierarchy, the retrieval system has no way to distinguish scope, priority, or relevance. An agent treating a temporary note the same as a permanent invariant will produce inconsistent behavior. As the number of entries grows, retrieval accuracy declines and drift accelerates.

Structured memory assigns every entry a **location**, a **type**, and a **weight**. This makes memory predictable and maintainable.

---

## Structural Model

### Wing

A wing is the top-level organizational unit. It defines a domain of concern.

Examples:
- `ros2` — all memory related to a ROS 2 project
- `architecture` — system design decisions and constraints
- `agent_behavior` — rules and instructions governing how agents operate
- `onboarding` — context intended for new contributors or sessions

A wing is not a tag. It is a scoping boundary. An agent working within a wing loads memory from that wing by default. Cross-wing retrieval is explicit and deliberate.

**Naming conventions:**
- Use lowercase identifiers
- Use underscores for multi-word names
- Keep wing names short and domain-specific

### Room

A room is a subdivision within a wing. Rooms group entries by sub-topic, lifecycle stage, or concern.

Examples within the `ros2` wing:
- `ros2/nodes` — per-node notes, constraints, and patterns
- `ros2/launch` — launch file conventions
- `ros2/interfaces` — message and service type decisions

Rooms should be narrow enough to be coherent but broad enough to avoid excessive fragmentation. If a room consistently has one or two entries, it may be too narrow.

### Entry

An entry is a single memory item within a room. It is the atomic unit of memory.

Every entry should have:
- **Type** — what kind of information it represents (see below)
- **Content** — the actual information
- **Created date** — when the entry was first written
- **Verified date** — when the entry was last confirmed accurate
- **Status** — `active`, `deprecated`, or `under_review`

---

## Entry Types

| Type | Description | Typical Weight |
|------|-------------|----------------|
| `invariant` | A rule or constraint that must never be violated | Highest |
| `decision` | An architectural or design choice with rationale | High |
| `pattern` | A recurring solution or approach to a class of problem | Medium |
| `note` | Contextual or supplementary information | Low |
| `deprecated` | Superseded content retained for traceability | Not surfaced by default |

Type determines how an agent should treat the entry during retrieval and reasoning. An agent must not override an `invariant` without explicit human instruction. A `note` may be deprioritized if context space is constrained.

---

## Hierarchy Summary

```
wing/
  room/
    entry (type: invariant | decision | pattern | note | deprecated)
```

Every entry has a complete address: `wing/room/entry-id`. This address is stable and referenceable across sessions.

---

## Design Guidance

- **Define wings before rooms.** The set of wings should reflect the actual domains of the project, not aspirational structure.
- **Do not create empty rooms in advance.** Rooms should emerge from actual entries, not be preallocated.
- **Entries should be atomic.** One entry should cover one concept. If an entry requires two distinct headings to organize its content, it is likely two entries.
- **Cross-wing references are links, not merges.** If a decision in the `architecture` wing affects the `ros2` wing, add a reference from the relevant room, not a copy of the entry.
