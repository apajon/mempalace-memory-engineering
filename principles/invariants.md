# Design Invariants

This document states the non-negotiable design invariants of the memory model described in
this repository. These are not agent behaviors and not authoring tips — they are properties
the memory system itself must satisfy. If an invariant here is violated, the model stops
being coherent.

Everything in [principles/rules.md](rules.md) derives from these invariants. Structural
definitions (wings, rooms, entries, retrieval order, scope) live in
[docs/architecture.md](../docs/architecture.md). Deduplication semantics live in
[docs/deduplication.md](../docs/deduplication.md).

These invariants apply to the methodology as documented here. They are not claims about any
specific implementation, tool, or deployment.

---

## I1 — Memory is scoped before it is retrieved

Every entry exists inside a wing and a room. Retrieval is defined relative to an active
scope, not against a global pool.

**Why it is an invariant.** Without scope, a project-specific rule and a shared platform
rule compete equally during retrieval. The agent has no principled way to choose. See
[docs/architecture.md §5 Scope Separation](../docs/architecture.md#5-scope-separation) and
[§6 Retrieval Model](../docs/architecture.md#6-retrieval-model).

**What violation looks like.** A flat memory store with no wing boundary. A "retrieve
everything relevant" query that crosses project and shared wings without the caller
declaring which one is active.

---

## I2 — Shared knowledge must not silently override project knowledge

Project wings encode local reality. Shared wings encode reusable conventions. When both
address the same topic, the project wing takes precedence — but only when that precedence
is explicitly marked.

**Why it is an invariant.** Silent override destroys trust in either layer. If a shared
rule can quietly win, project rules are not reliable. If a project rule can quietly win,
shared conventions drift per project.

**What violation looks like.** A retrieval implementation that merges both wings and picks
the "best match" by similarity. A project entry that contradicts a shared entry without
declaring itself a local override.

See [docs/architecture.md §7 Conflict Handling](../docs/architecture.md#7-conflict-handling).

---

## I3 — Project-specific overrides must be explicit

A project entry that diverges from a shared entry must say so and state why. Implicit
divergence is indistinguishable from drift.

**Example of an explicit override:**

```
type: decision
label: local-override
status: active

This project does not follow the shared `ros2/lifecycle` activation sequence
because the hardware driver requires a custom init step before activate().
```

**What violation looks like.** A project entry that restates a shared rule with a small
change, no `local-override` marker, and no rationale. A reader cannot tell whether this is
an intentional deviation, a stale copy, or a mistake.

---

## I4 — Memory writes require prior memory reads

Before writing a new entry, the existing entries in the target room — and, for shared
concerns, the target shared wing — must be consulted. Writing without reading produces
duplicates and contradictions.

**Why it is an invariant.** Duplication is a retrieval failure, not a stylistic problem.
Two entries that say nearly the same thing with different wording force the agent to
reconcile them on every retrieval. See
[docs/deduplication.md §1](../docs/deduplication.md#1-why-deduplication-matters) and
[§3 Wing and Room Scoped Comparison](../docs/deduplication.md#3-wing-and-room-scoped-comparison).

**What violation looks like.** Creating an entry on a topic without first checking the
room. A "write-first, search-later" workflow.

---

## I5 — Durable memory must be non-obvious, reusable, and likely to matter again

An entry earns its place only if it meets all three:

- **Non-obvious** — not trivially recoverable by reading the code or the public docs of the
  framework.
- **Reusable** — will apply to more than the single task that surfaced it.
- **Likely to matter again** — the cost of forgetting it is real.

**Why it is an invariant.** Memory that fails any of the three pollutes retrieval. Obvious
facts waste context. One-shot facts age into noise. Unlikely-to-recur facts occupy space
that future signal needs.

**What violation looks like.** Session transcripts persisted as entries. Restated
framework documentation. Notes about a single bug that is already fixed and will not
recur.

See [docs/architecture.md §8 Persistence Rules](../docs/architecture.md#8-persistence-rules).

---

## I6 — Memory unavailability must not block development work

The memory backend is an input, not a precondition. When it is unavailable, degraded, or
uncertain, work continues through documented fallbacks.

**Why it is an invariant.** A memory system that can halt development is a liability.
Agents and humans both depend on the ability to proceed with local sources (repo docs,
README, code search) when the structured store is not reachable.

**What violation looks like.** An agent that refuses to answer or edit because memory
retrieval returned an error. A workflow that requires memory writes to succeed before a
change is considered done.

See [docs/architecture.md §9 Graceful Degradation](../docs/architecture.md#9-graceful-degradation).

---

## I7 — Obsolete memory is maintenance debt

An entry that no longer matches reality is worse than no entry. It actively misleads
retrieval. Obsolescence must be marked, not ignored, and supersession must be explicit.

**Why it is an invariant.** The trust contract of memory is that what is retrieved is
usable. A stale `active` entry breaks that contract. Silent deletion breaks traceability.

**What violation looks like.** Overwriting an entry to "fix" it without recording that a
prior rule existed. Leaving superseded entries `active` alongside their replacements.
Deleting an entry instead of marking it `deprecated`.

---

## I8 — One entry contains one durable rule or fact

An entry is atomic. If it needs two headings to organize its content, it is two entries.
If it mixes a rule with an unrelated anti-pattern, it is two entries.

**Why it is an invariant.** Atomicity is what makes entries referenceable, deprecatable,
and enrichable. Compound entries cannot be cleanly superseded: deprecating them throws out
still-valid content; editing them silently changes rules that other entries may
cross-reference.

**What violation looks like.** An entry titled "Lifecycle and message handling" that
bundles an activation-state rule with a subscriber-queue pattern. Each deserves its own
address.

---

## Scope of These Invariants

These invariants constrain the memory model. They are not claims about agent reasoning or
natural language understanding. They assume a disciplined author — human or agent — that
can decide whether a given piece of content qualifies for persistence, which wing it
belongs in, and whether it duplicates existing content.

When tooling enforces some of these invariants (for example, a write API that rejects
entries without a `created_at` field), that is a convenience. The invariants hold whether
or not tooling enforces them.
