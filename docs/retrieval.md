# Retrieval: Order, Scope, and Priority

Retrieval is not search. Retrieval is a structured process by which an agent assembles relevant context from memory before producing output or making a decision. This document defines how retrieval should be ordered and scoped.

---

## Retrieval Order

When an agent retrieves memory for a given task, entries should be applied in this order:

1. **Invariants** — Retrieved and applied unconditionally, regardless of task context. These represent constraints the agent must never violate.
2. **Decisions** — Scoped to the active wing and room. Applied after invariants to provide context-specific direction.
3. **Patterns** — Matched to the current task or problem class. Applied after decisions to provide reusable approaches.
4. **Notes** — Supplementary context. Applied last and deprioritized when context space is limited.

`deprecated` entries are excluded from retrieval unless explicitly requested by the user or agent instruction.

This ordering ensures that high-priority constraints are never overridden by lower-priority contextual notes.

---

## Scope

Retrieval is scoped to the **active wing** by default.

An agent working on a ROS 2 task operates in the `ros2` wing. It retrieves entries from that wing without loading unrelated wings. This prevents context pollution and ensures that agent behavior is predictable within a domain.

Cross-wing retrieval is allowed but must be **explicit**. Agent instructions or user prompts that reference another wing cause retrieval to extend to that wing. This should be reflected in agent instructions (see `templates/agent_instructions_template.md`).

---

## Filtering

Within a wing, retrieval may be further scoped:

- **By room** — Retrieve only entries from a specific room when the task is tightly scoped.
- **By type** — Retrieve only invariants, or only decisions, when the agent is performing a specific reasoning task.
- **By status** — Only `active` entries are included in default retrieval. Entries with status `under_review` may be included with a note that they are unconfirmed.

---

## Context Window Considerations

Retrieved entries consume context window space. When memory volume is high:

- Invariants are always included.
- Decisions are included in full.
- Patterns are summarized or truncated if needed.
- Notes are included only if directly relevant to the current task.

Agent instructions should specify how to handle context pressure explicitly rather than relying on implicit truncation behavior.

---

## Retrieval Failures

Retrieval failures occur when:

- The agent draws on an entry that does not exist (hallucination of memory)
- The agent ignores a relevant entry that does exist
- The agent applies an entry outside its intended scope

These failures are symptoms of poor memory design, not retrieval system bugs. Fixing them requires reviewing entry types, scope assignments, and agent instructions — not just re-running retrieval.
