# Persistence: Durability and Ownership of Memory

Memory entries are artifacts. They should be treated with the same care as code, configuration, and architecture documents.

---

## Core principle

Memory should survive:

- agent restarts
- model upgrades or replacements
- tooling changes
- team changes over time

If knowledge only exists in a live session or in an agent's current context window, it is temporary state, not persisted memory.

---

## What must be persisted

The following types are worth storing durably:

- **Invariants** — because losing a system constraint changes agent behavior
- **Decisions** — because design rationale disappears quickly if not recorded
- **Patterns** — because repeated engineering knowledge is expensive to rediscover

Notes may also be persisted, but only when they are durable enough to matter later.

---

## Storage backend

The backend handles mechanics such as writing, indexing, and retrieving entries.

This repository does not require a specific backend. It does require a few practical properties:

- entries remain readable outside the agent runtime
- metadata is preserved
- stored memory is not confused with transient session context

---

## Version history

Entries that shape system behavior should keep enough history to explain what changed.

At minimum, important entries should carry `created_at` and usually `verified_at`.

When an entry changes substantially, the previous version should remain available through:

- backend version history
- or an explicit deprecated entry retained for traceability

Invariants and decisions benefit most from this.

---

## Human readability

Persisted memory must remain understandable to a human without requiring an agent to interpret it.

That matters because:

1. humans need to audit and correct stored memory
2. clear writing also improves machine retrieval

If an entry is opaque to a human reviewer, it is not good durable memory.

---

## Persistence anti-patterns

- **Session-only memory** — useful in the moment, but not durable
- **Implicit memory** — something the agent "remembers" without a stored record
- **Unverified persistence** — assuming an entry was stored without confirming it exists

If the knowledge must matter later, make sure it is actually written to durable storage and can be found again.