# MemPalace Memory Engineering — Copilot Instructions

Guidelines for working in this repository. This is a **methodology and reference repo** for
designing long-term memory in AI systems. The artifacts here are Markdown documents —
principles, templates, examples, and docs. Changes have downstream impact on any agent
that consumes these conventions.

See [README.md](../README.md) for project overview and core concepts.

---

## 1. Think Before Writing

**Don't assume. Surface ambiguity. Name the tradeoffs.**

Before proposing or editing any content:

- State which **wing**, **room**, and **entry type** the change targets. If uncertain, ask.
- If multiple entry types could apply (e.g., `invariant` vs. `decision`), present the
  distinction — don't pick silently.
- If a simpler formulation exists, say so. Push back when warranted.
- If the scope is unclear (global vs. wing-scoped), stop and surface the question.

---

## 2. Minimum Viable Entry

**Write the least content that fully captures the constraint. Nothing speculative.**

- No content beyond what the task requires.
- No "related thoughts" or loosely connected patterns appended to an entry.
- No preemptive entries for constraints that haven't been established yet.
- If an entry can be stated in 3 lines, don't write 10.

Ask: *"Would a reviewer reading this entry know exactly what it enforces, and nothing more?"*
If not, tighten it.

---

## 3. Surgical Edits

**Touch only what you must. Preserve the structure around your changes.**

When editing existing docs or entries:

- Don't reformat, re-word, or restructure sections you weren't asked to touch.
- Match the existing style and terminology (e.g., `must`, `must not`, `should`).
- If you notice a drift, inconsistency, or stale entry elsewhere, **mention it — don't fix it**
  unless explicitly asked.

When your changes create orphans:
- Remove cross-references or links that YOUR changes made invalid.
- Don't clean up pre-existing broken references unless asked.

---

## 4. Goal-Driven Authoring

**Every change should have a verifiable outcome.**

Transform tasks into checkable goals:

- "Add an invariant" → "The entry states a specific constraint, scoped to a wing, with a
  verifiable consequence of violation."
- "Update a pattern" → "The updated entry does not contradict any invariant in the same wing."
- "Deprecate a decision" → "The old entry is marked `deprecated`, references its replacement,
  and is not surfaced by default retrieval."

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

---

## Memory Engineering Conventions

**These apply to all content in this repo.**

- **Entry types**: `invariant` > `decision` > `pattern` > `note`. See [README.md](../README.md#entry-types)
  for type definitions.
- **Retrieval order**: Invariants apply first. Lower-weight entries do not override them.
  See [docs/retrieval.md](../docs/retrieval.md).
- **Draft before activate**: Proposed entries start as `status: draft`. No entry becomes
  `active` without explicit human approval. This is non-negotiable for `invariant` and
  `decision` entries.
- **Metadata is required**: Every entry must include `type`, `status`, and `created_at`.
  Missing metadata is a defect, not an omission.
- **Supersession is explicit**: Replaced entries are marked `deprecated` with a reference to
  their replacement. Silent deletion is not permitted. See
  [principles/rules.md § 6](../principles/rules.md#6-when-to-mark-obsolete) and
  [principles/invariants.md § I7](../principles/invariants.md#i7--obsolete-memory-is-maintenance-debt).
- **Scope invariants correctly**: A wing-scoped invariant does not apply globally. If a
  constraint must apply across all wings, flag it for placement in a `global` wing.
  See [principles/invariants.md § I1](../principles/invariants.md#i1--memory-is-scoped-before-it-is-retrieved).
- **No duplication**: Before drafting a new entry, check for existing coverage. Propose an
  update over a new entry if overlap exists. See
  [principles/rules.md § 5](../principles/rules.md#5-when-to-enrich-instead-of-creating).

---

**Accountability check:** Every changed line should trace to the user's request. Entries that
conflict with an invariant must surface the conflict before proceeding. Ambiguity should be
named, not resolved silently.
