# Rules: Behavioral Standards for Agents Using Memory

These rules define how agents should behave with respect to structured memory. They apply across all wings and all task types unless explicitly overridden by a human for a specific session.

---

## Retrieval Rules

**R1 — Load invariants first.**
Before producing output or making a decision, the agent must retrieve and apply invariants from the active wing. Invariants are not optional context; they are constraints.

**R2 — Scope retrieval to the active wing.**
Retrieval is limited to the active wing by default. Cross-wing retrieval requires explicit instruction. The agent must not silently pull entries from unrelated wings.

**R3 — Exclude deprecated entries.**
Entries with status `deprecated` must not be included in standard retrieval. They may be accessed when explicitly requested, but they must not influence default agent behavior.

**R4 — Surface retrieval gaps.**
If the agent cannot find relevant memory for a task, it should note this rather than proceeding silently. A missing entry may indicate that memory has not been written yet, not that no constraint exists.

---

## Authoring Rules

**R5 — Draft, do not activate.**
The agent may draft new memory entries but must not mark them `active` without human approval. This applies to all entry types, with particular importance for `invariant` and `decision` entries.

**R6 — Be specific.**
Memory entries authored by agents must be specific and actionable. The agent must not write vague entries. If the content cannot be made specific, the agent should flag the ambiguity for human resolution.

**R7 — Do not duplicate.**
Before drafting a new entry, the agent should check whether an existing entry already covers the topic. If a related entry exists, the agent should note the relationship and propose an update rather than creating a new entry.

**R8 — Include metadata.**
Drafted entries must include `type`, `status: draft`, and `created_at`. The agent must not omit these fields.

---

## Behavioral Rules

**R9 — Invariants override convenience.**
An agent must not violate an invariant for the sake of brevity, speed, or user preference. If a user request conflicts with an invariant, the agent must surface the conflict explicitly and wait for human resolution.

**R10 — Decisions are stable until revised.**
An agent must not propose alternatives to active decisions unless explicitly asked to reconsider. Existing decisions represent concluded reasoning. Re-litigating them in every session wastes context and creates noise.

**R11 — Flag drift, do not paper over it.**
If the agent detects that an entry does not match the current system state, it must flag this discrepancy explicitly. It must not silently use the current system state as if it were correct memory, nor must it silently ignore the discrepancy and proceed using stale memory.

**R12 — Attribute decisions to memory.**
When the agent's behavior is driven by a memory entry, it should be able to state which entry is the source. "I am following the `no-blocking-callbacks` invariant in `ros2/invariants`" is more useful than "I am following best practices."

---

## Memory Governance Rules

**R13 — Humans approve invariants.**
No invariant may become `active` without explicit human approval. The agent may identify the need for an invariant and draft it, but activation requires human sign-off.

**R14 — Supersession is explicit.**
When an entry is superseded, the old entry must be marked `deprecated` with a reference to the replacement. Silent deletion is not permitted.

**R15 — Memory is not the context window.**
The agent must not treat information present only in the current session's context window as persistent memory. Information that should persist must be written to the memory backend.
