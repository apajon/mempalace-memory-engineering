# Retrieval: Order, Scope, and Priority

Retrieval is not just search. It is how an agent assembles the right memory before it answers a question, makes a change, or takes a decision.

---

## Retrieval order

Apply entries in this order:

1. **Invariants** — hard constraints
2. **Decisions** — current design direction
3. **Patterns** — reusable ways of solving the problem
4. **Notes** — supporting context

Entries with `status: deprecated` stay out of default retrieval unless explicitly requested.

---

## Scope

Retrieval is scoped to the active wing by default.

If an agent is working on a ROS 2 task, it should start with the `ros2` wing rather than searching across unrelated memory.

Cross-wing retrieval is allowed, but it should be explicit.

---

## Filtering

Within a wing, retrieval can be narrowed:

- **By room**
- **By type**
- **By status**

Default retrieval should focus on `active` entries and include `under_review` entries only when their uncertainty is made clear.

---

## Context window pressure

Retrieved memory uses context space, so degradation should be deliberate.

When context gets tight:

- keep invariants first
- keep decisions in full
- summarize patterns if needed
- include notes only when directly relevant

---

## Retrieval failures

Retrieval usually fails in one of three ways:

- the agent uses an entry that does not exist
- the agent ignores an entry that does exist
- the agent applies an entry outside its scope

When that happens, the fix is usually to review:

- entry types
- scope assignment
- room placement
- status accuracy
- instruction-file behavior