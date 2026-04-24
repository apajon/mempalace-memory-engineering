# Design Invariants

These are the non-negotiable properties of the memory model used in this repository. They are not style tips and not agent behaviors. They are the structural conditions that keep the model coherent.

See [principles/rules.md](rules.md) for the practical rules built on top of them.

---

## I1 — Memory is scoped before it is retrieved

Every entry belongs to a wing and a room.

**Why this matters:** without scope, project and shared rules compete on equal footing.

**What violation looks like:** a flat memory store or a retrieval request that searches everything without declaring active scope.

---

## I2 — Shared knowledge must not silently override project knowledge

Project wings represent local reality. Shared wings represent reusable conventions.

**Why this matters:** silent overrides destroy trust in both layers.

**What violation looks like:** retrieval that picks the "best match" across project and shared wings with no explicit override rule.

---

## I3 — Project-specific overrides must be explicit

If a project entry diverges from a shared rule, it should say so and explain why.

**Why this matters:** implicit divergence looks exactly like drift.

**What violation looks like:** a project entry that restates a shared rule with one small change and no explanation.

---

## I4 — Memory writes require prior memory reads

Before writing a new entry, check the target room first, and the shared wing if the topic may already live there.

**Why this matters:** writing before reading is the fastest route to duplicates.

**What violation looks like:** a workflow that creates entries first and searches later.

---

## I5 — Durable memory must be non-obvious, reusable, and likely to matter again

An entry earns durable storage only if it is:

- non-obvious
- reusable
- likely to matter again

**Why this matters:** anything less turns into retrieval noise.

**What violation looks like:** session transcripts, restated framework docs, or one-off facts.

---

## I6 — Memory unavailability must not block development work

The memory backend is an input, not a precondition.

**Why this matters:** otherwise memory becomes a single point of failure.

**What violation looks like:** an agent that refuses to continue because retrieval failed.

---

## I7 — Obsolete memory is maintenance debt

An entry that no longer matches reality is worse than no entry.

**Why this matters:** a stale `active` entry breaks trust in the whole system.

**What violation looks like:** leaving superseded entries active, deleting old entries without a trace, or silently overwriting a rule.

---

## I8 — One entry contains one durable rule or fact

Entries should stay atomic.

**Why this matters:** atomic entries are easier to reference, review, supersede, and retrieve.

**What violation looks like:** one entry that mixes unrelated rules or patterns.

---

## Scope of these invariants

These invariants constrain the memory model itself. Tooling may enforce some of them, but the methodology should still hold even when tooling does not.