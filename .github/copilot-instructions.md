# MemPalace Memory Engineering — Copilot Instructions

This repository is a methodology and reference repo for long-term memory design in AI systems. Most files here are Markdown documents, templates, and examples. Changes may affect downstream agents that rely on these conventions.

See [README.md](../README.md) for the overview and core concepts.

---

## 1. Think before writing

Do not assume scope or entry type.

Before proposing or editing content:

- state which **wing**, **room**, and **entry type** the change targets
- if more than one entry type could fit, explain the distinction instead of picking silently
- prefer the simplest wording that still captures the constraint
- if scope is unclear, stop and surface that ambiguity

---

## 2. Minimum viable entry

Write the smallest amount of content that fully captures the rule.

- no speculative content
- no loosely related appendices
- no preemptive entries for constraints that are not established yet
- if an entry can be three lines long, do not make it ten

---

## 3. Surgical edits

Touch only what the task requires.

When editing existing docs or entries:

- do not rewrite unrelated sections
- match the repository's existing terminology such as `must`, `must not`, and `should`
- if you notice another inconsistency, mention it rather than fixing it unless the task requires it

If your change invalidates a cross-reference, repair that reference. Do not clean up unrelated broken links opportunistically.

---

## 4. Goal-driven authoring

Every change should lead to a checkable outcome.

Examples:

- adding an invariant means the entry states a clear constraint with a verifiable consequence of violation
- updating a pattern means the updated text does not contradict active invariants in the same scope
- deprecating a decision means the old entry is marked `deprecated`, references its replacement, and stays out of default retrieval

For multi-step work, state a brief plan with one verification check per step.

---

## Memory engineering conventions

- **Entry types:** `invariant` > `decision` > `pattern` > `note`
- **Retrieval order:** invariants apply first
- **Draft before activate:** proposed entries start as `status: draft`; no entry becomes `active` without explicit human approval
- **Metadata is required:** every entry must include `type`, `status`, and `created_at`
- **Supersession is explicit:** replaced entries are marked `deprecated` and reference their replacement
- **Scope matters:** a wing-scoped invariant does not become global by accident
- **No duplication:** check for existing coverage before drafting a new entry

See:

- [README.md](../README.md#entry-types)
- [docs/retrieval.md](../docs/retrieval.md)
- [principles/rules.md](../principles/rules.md#6-when-to-mark-obsolete)
- [principles/invariants.md](../principles/invariants.md#i7--obsolete-memory-is-maintenance-debt)

---

**Accountability check:** every changed line should trace back to the task. If a proposed change conflicts with an invariant, surface the conflict instead of resolving it silently.