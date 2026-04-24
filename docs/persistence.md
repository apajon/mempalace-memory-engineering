# Persistence: Durability and Ownership of Memory

Memory entries are artifacts. They should be treated with the same care as source code, configuration files, or design documents. This document defines expectations for how memory persists and what makes persistence reliable.

---

## Core Principle

Memory must survive:

- Agent session restarts
- Model changes or upgrades
- Tooling changes (editor, IDE, MCP client)
- Team membership changes

If memory exists only in a session or in the context window of a running agent, it is not persisted memory — it is ephemeral state. Ephemeral state should not be treated as memory.

---

## What Must Be Persisted

The following entry types must always be persisted to durable storage:

- **Invariants** — These define the constraints of the system. Loss of an invariant means an agent may operate without knowing a rule exists.
- **Decisions** — These record why the system is the way it is. Loss of a decision means rationale disappears, making future decisions less informed.
- **Patterns** — These encode accumulated engineering knowledge. They are worth persisting because they represent repeated validation.

Notes may be persisted but are lower priority. Temporary notes that are session-scoped should be explicitly labeled as such and not stored in wings intended for durable memory.

---

## Storage Backend

Persistence mechanics — writing, reading, and indexing entries — are handled by the storage backend. If using MemPalace, the backend manages file-level persistence and MCP-accessible retrieval.

This repository does not prescribe a specific backend. The methodology is backend-agnostic. What matters is that:

- The backend stores entries in a format that is human-readable outside the agent context
- The backend preserves entry metadata (type, dates, status)
- The backend is not the agent's context window

---

## Version History

Entries that define system behavior should carry version history or at minimum a `created_at` and `verified_at` timestamp. When an entry is updated substantially, the previous version should be accessible, either through the backend's versioning support or by maintaining the deprecated original entry.

This is not a requirement for all entries. Notes and low-weight patterns do not require version history. Invariants and decisions do.

---

## Human Readability

Memory entries must be readable by a human without using an agent. If an entry is only interpretable by an AI, it is not truly persisted — it is opaque state. Entries should be written in clear, direct language that a human engineer can understand and verify.

This requirement serves two purposes:
1. It allows humans to audit and correct memory.
2. It forces entries to be well-formed enough that an AI will also interpret them correctly.

---

## Persistence Anti-Patterns

- **Session-only memory** — Memory that only exists in a running agent's context window is not persistent. Do not rely on it across sessions.
- **Implicit memory** — Memory that exists only because an agent "remembers" it from previous interactions, without being written to a backend, is not persistent.
- **Unverified persistence** — Assuming an entry was saved without confirming it exists in the backend is a common source of lost memory.
