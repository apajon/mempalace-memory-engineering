# MemPalace Memory Engineering

![Memory Engineering](https://img.shields.io/badge/domain-memory%20engineering-0969da)
![Status](https://img.shields.io/badge/status-reference%20guide-2ea043)
![License](https://img.shields.io/github/license/apajon/mempalace-memory-engineering)

> A practical guide to designing long-term memory for AI systems.

Most AI memory fails because it is treated as a dump.

This repository treats memory as an engineering artifact: scoped, typed, reviewed, deduplicated, and maintained.

---

## Why This Repository Exists

As AI-assisted engineering grows, agents accumulate context. Without structure, that memory becomes hard to trust.

- **Flat memory breaks down.** Hard rules, decisions, and loose notes all look the same.
- **Drift is easy to miss.** Old entries stay in circulation until someone reviews them.
- **Retrieval has a cost.** Irrelevant memory takes space away from useful memory.

The core idea is simple: memory should be engineered, not accumulated.

---

## Core Concepts

### Wings and Rooms

Memory is organized as:

- **Wing** — the top-level scope
- **Room** — a subject area inside a wing
- **Entry** — one durable memory item inside a room

This keeps project knowledge, shared knowledge, and low-value notes from getting mixed together.

### Entry Types

| Type | Purpose |
|------|---------|
| `invariant` | A rule that must not be violated |
| `decision` | A design or architecture choice with rationale |
| `pattern` | A reusable solution or practice |
| `note` | Supporting context with lower priority |

Entries also have a **status**: `draft`, `active`, `under_review`, or `deprecated`.

### Retrieval Order

Apply entries in this order:

1. `invariant`
2. `decision`
3. `pattern`
4. `note`

Entries with `status: deprecated` stay out of default retrieval unless requested explicitly.

### Deduplication and Drift Control

Healthy memory needs maintenance:

- **Deduplication** checks whether a rule already exists before a new entry is added.
- **Drift control** identifies entries that are stale or no longer accurate.
- **Supersession** replaces entries explicitly instead of silently deleting them.

In practice, entries should carry `created_at`, usually `verified_at`, and a clear status.

### Persistence Rules

Durable memory is not the same as session context.

- Important rules and decisions should survive restarts and tool changes.
- Stored entries should remain readable to humans.
- Persistence belongs to the backend. This repository focuses on structure and governance.

---

## What This Repository Is

This is a methodology and reference repository for long-term memory in AI-assisted engineering workflows. It documents:

- how to structure memory with wings, rooms, and typed entries
- how to write agent instructions that use memory safely
- patterns that scale
- anti-patterns that make memory brittle
- templates for consistent entry authoring

---

## What This Repository Is Not

- It is **not** a runtime system.
- It is **not** a plugin or extension.
- It is **not** a replacement for MemPalace.
- It is **not** official MemPalace documentation.

---

## Start Here

If you are new:

1. Read [docs/architecture.md](docs/architecture.md) to understand the wing / room / entry model.
2. Copy [templates/agent_instructions_template.md](templates/agent_instructions_template.md) as a starting point for agent behavior.
3. Read [examples/basic_two_wing_model.md](examples/basic_two_wing_model.md) to see the model used in practice.
4. Add deduplication rules from [docs/deduplication.md](docs/deduplication.md) once memory starts growing.

If you already know the model:

- Read [docs/retrieval.md](docs/retrieval.md) for retrieval order, fallback behavior, and context-window strategy.
- Read [principles/rules.md](principles/rules.md) for operational authoring rules.
- Use [templates/memory_entry_template.md](templates/memory_entry_template.md) when drafting durable entries.

---

## Quick Navigation

Start here:

- [README.md](README.md) — overview and core concepts
- [docs/architecture.md](docs/architecture.md) — the wing / room / entry model
- [docs/retrieval.md](docs/retrieval.md) — retrieval order, scope, and priority

Core methodology:

- [docs/deduplication.md](docs/deduplication.md) — deduplication and drift control
- [docs/persistence.md](docs/persistence.md) — what durable memory must preserve
- [docs/maintenance.md](docs/maintenance.md) — lifecycle and upkeep
- [docs/anti_patterns.md](docs/anti_patterns.md) — common failure modes

Rules and templates:

- [principles/invariants.md](principles/invariants.md) — non-negotiable constraints
- [principles/rules.md](principles/rules.md) — practical authoring rules
- [templates/agent_instructions_template.md](templates/agent_instructions_template.md) — starting point for agent instructions
- [templates/memory_entry_template.md](templates/memory_entry_template.md) — starting point for memory entries

Examples:

- [examples/basic_two_wing_model.md](examples/basic_two_wing_model.md) — minimal two-wing setup
- [examples/ros2_agent_instructions.md](examples/ros2_agent_instructions.md) — worked agent-instructions example

---

## Relationship to MemPalace

[MemPalace](https://github.com/apajon/mempalace) is the runtime layer. It handles storage, persistence, and MCP-based access.

This repository covers the design side: what to store, how to organize it, how agents should read it, and how to maintain it over time.

Use MemPalace to store memory. Use this repository to design memory that stays useful.

---

## Relationship to mempalace-mcp-bridge

The [mempalace-mcp-bridge](https://github.com/apajon/mempalace-mcp-bridge) handles integration: configuration, transport, and connectivity.

This repository handles methodology: what belongs in memory, what does not, and how to keep it coherent as it grows.

---

## Repository Structure

```text
docs/               Core methodology docs
  architecture.md   Wing / room / entry model
  retrieval.md      Retrieval order, scope, and priority
  deduplication.md  Deduplication and drift control
  persistence.md    Durability and storage expectations
  maintenance.md    Lifecycle and upkeep
  anti_patterns.md  Common failure modes to avoid

examples/           Worked examples
  basic_two_wing_model.md
  ros2_agent_instructions.md

principles/         Foundational constraints and operating rules
  invariants.md
  rules.md

templates/          Reusable starting points
  agent_instructions_template.md
  memory_entry_template.md
```

---

## License

See [LICENSE](LICENSE).
