# MemPalace Memory Engineering

> Design structured, scalable, and usable long-term memory for AI systems.

---

## Why This Exists

As AI-assisted engineering workflows grow more complex, agents accumulate more context. Without deliberate structure, memory degrades:

- **Flat memory becomes unreliable.** When all entries exist at the same level without hierarchy or taxonomy, agents cannot distinguish between a one-time note and a permanent architectural invariant. Retrieval quality degrades as volume increases.
- **Memory drift occurs silently.** Outdated entries contradict current ones. Conflicts go undetected. The agent starts drawing on stale or superseded information as if it were still current.
- **Retrieval is not free.** Context windows are finite. When memory is unstructured, the wrong entries surface at the wrong time, crowding out entries that actually matter.

Memory should be engineered, not accumulated.

---

## Core Concepts

### Wings and Rooms

Memory is organized hierarchically:

- **Wing** — A top-level domain of concern (e.g., `ros2`, `architecture`, `agent_behavior`). Wings define scope. An agent operating in a wing only loads memory relevant to that wing unless explicitly instructed otherwise.
- **Room** — A subdivision within a wing. Rooms group entries by sub-topic or lifecycle stage (e.g., `ros2/nodes`, `ros2/launch`, `architecture/decisions`).
- **Entry** — A single memory item within a room. Entries have types, metadata, and content.

This structure prevents the flat-memory problem by requiring that every entry have a known location and purpose.

### Entry Types

Not all memory is equal. Common types include:

| Type | Purpose |
|------|---------|
| `invariant` | A rule that must never be violated (e.g., "always use `rclcpp::spin` in the main node thread") |
| `decision` | An architectural or design decision with rationale |
| `pattern` | A recurring solution or approach |
| `note` | Contextual information, lower weight |
| `deprecated` | Superseded content kept for traceability |

Typed entries allow retrieval systems and agents to apply weight and priority appropriately.

### Retrieval Order

When an agent needs to resolve a question or produce output, it should retrieve memory in this order:

1. **Invariants** — Applied first, unconditionally.
2. **Decisions** — Scoped to the active wing and room.
3. **Patterns** — Matching the current task or context.
4. **Notes** — Supplementary context, used last.

Entries marked `deprecated` are not surfaced unless explicitly requested.

### Deduplication and Drift Control

Memory engineering requires active maintenance of content integrity:

- **Deduplication** is the process of identifying entries that cover the same ground and merging or removing redundant ones. Deduplication should be performed on a schedule, not only reactively.
- **Drift control** means detecting when an entry no longer reflects the current system state. Entries should carry a `verified_at` or equivalent timestamp. Entries that have not been reviewed within a defined window should be flagged.
- **Supersession** is explicit: when a decision changes, the old entry should be marked `deprecated` with a reference to the replacement, not silently deleted.

### Persistence Rules

Memory entries should be treated as artifacts, not ephemeral notes:

- Entries that define system behavior or constraints must be persisted with version history.
- Entries should be reviewable by humans outside the agent context.
- Persistence should not depend on a single session or runtime. Memory must survive agent restarts, model changes, and tooling updates.
- Storage and access are handled by the underlying memory backend (e.g., MemPalace). Structure and methodology are what this repository provides.

---

## What This Repository Is

This is a **methodology and reference repository** for designing long-term memory in AI-assisted engineering workflows. It documents:

- How to structure memory using wings, rooms, and typed entries
- How to write agent instructions that use memory correctly
- Patterns that work well at scale
- Anti-patterns that produce brittle or unreliable memory behavior
- Templates for consistent entry and instruction authoring

---

## What This Repository Is Not

- It is **not** a runtime system. It does not execute or serve memory.
- It is **not** a plugin or extension for any specific tool.
- It is **not** a MemPalace replacement or fork.
- It is **not** official MemPalace documentation.

---

## Relationship to MemPalace

[MemPalace](https://github.com/apajon/mempalace) provides **storage and MCP access** — the runtime layer that allows agents to read and write structured memory. It handles the mechanics of persistence, retrieval, and protocol.

This repository provides **structure and methodology** — how to design what goes into that storage, how to organize it, how to write against it, and how to keep it healthy over time.

Use MemPalace to store memory. Use this repository to design it.

---

## Relationship to mempalace-mcp-bridge

The [mempalace-mcp-bridge](https://github.com/apajon/mempalace-mcp-bridge) handles **setup and runtime integration** — connecting agents and tools to the MemPalace backend via the Model Context Protocol. It is concerned with wiring, configuration, and transport.

This repository handles **memory design patterns** — what to put in memory, how to structure it, and how agents should behave with respect to it. These are complementary concerns. The bridge gets memory flowing; this repository determines what that memory should look like.

---

## Repository Structure

```
docs/               Core documentation
  architecture.md   Wing/room/entry model
  retrieval.md      How retrieval order and scoping work
  deduplication.md  Deduplication and drift control
  persistence.md    Persistence rules and durability expectations
  maintenance.md    Memory lifecycle and maintenance schedule
  anti_patterns.md  Common failure modes to avoid

examples/           Worked examples
  basic_two_wing_model.md      A minimal two-wing setup
  ros2_agent_instructions.md   Agent instructions for a ROS 2 project

principles/         Foundational rules
  invariants.md     What invariants are and how to define them
  rules.md          Behavioral rules for agents using memory

templates/          Reusable starting points
  agent_instructions_template.md
  memory_entry_template.md
```

---

## License

See [LICENSE](LICENSE).
