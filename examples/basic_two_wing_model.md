# Example: Basic Two-Wing Memory Model

A minimal, concrete example of organizing long-term memory around **two wings**:
one scoped to a project, one shared across projects. The goal is to keep
project-specific rules separated from reusable platform knowledge, so retrieval
stays predictable and shared knowledge does not have to be duplicated.

For the underlying methodology, see [docs/architecture.md](../docs/architecture.md)
and [docs/retrieval.md](../docs/retrieval.md).

---

## 1. Setup: Two Wings, Three Rooms

```
myapp/              ← project wing (scoped to one codebase)
  architecture/     ← room: project-specific design rules
  anti-patterns/    ← room: confirmed mistakes to avoid in this project

react/              ← shared wing (reusable across projects)
  patterns/         ← room: general React knowledge
```

Why two wings?

- The **project wing** (`myapp`) holds knowledge that only makes sense for this
  codebase: local conventions, project-specific overrides, anti-patterns that
  were actually observed here.
- The **shared wing** (`react`) holds knowledge that is reusable across any
  React project. Keeping it separate prevents the "Project Island" problem
  where the same rule is re-stated (and slowly drifts) inside every project
  wing.

Start with these three rooms. Add more only when enough real content justifies
them (see [Optional: Operational Rooms](#4-optional-operational-rooms)).

---

## 2. Sample Entries

Each entry below includes a short note on **why it belongs where it belongs**.
The goal is to make scope placement explicit, not implicit.

### 2.1 Architecture rule — project wing

**Wing:** `myapp` · **Room:** `architecture`

```
type: invariant
label: architecture-rule
status: active
created_at: 2026-03-10

All data fetching must go through the repository layer (src/repositories/).
Components must never call the API client directly.

Rationale: keeps components testable and isolates network concerns.
```

Why here: this rule is specific to `myapp`'s layering. It references a concrete
path (`src/repositories/`) that does not exist outside this project. It is
therefore wrong to place it in the shared `react` wing.

---

### 2.2 Anti-pattern — project wing

**Wing:** `myapp` · **Room:** `anti-patterns`

```
type: invariant
label: anti-pattern
status: active
created_at: 2026-03-18

Do not store derived state in Redux when it can be computed from existing state.
This caused a stale-data bug in the cart total after a coupon was applied.
Compute totals in selectors instead.

Rationale: derived state in Redux drifts from its inputs and creates
stale-read bugs that are hard to trace.
```

Why here: the rule is phrased as a **must-not**, and it is rooted in a concrete
incident observed in `myapp`. Anti-patterns live in their own room so they can
be surfaced explicitly during code review without being drowned in general
architecture rules.

Note on type: anti-patterns are usually modeled as `invariant` entries
(something that must never happen). The `anti-pattern` label marks the
*intent* of the rule — see
[docs/architecture.md § Entry Types](../docs/architecture.md#entry-types).

---

### 2.3 Shared knowledge — shared wing

**Wing:** `react` · **Room:** `patterns`

```
type: pattern
label: reusable-pattern
status: active
created_at: 2026-01-20

React Context is appropriate for low-frequency global state (theme, locale, auth).
Do not use Context for high-frequency updates (e.g., form fields, animations) —
use local state or a dedicated store instead.
```

Why here: the rule applies to **any** React codebase, not only `myapp`. It is
not coupled to project-specific paths, components, or domain concepts. If it
were placed in `myapp/architecture` it would either need to be copied into
every React project wing, or one of the copies would slowly diverge.

---

## 3. Retrieval Order

Before modifying a data-fetching component, an agent should query memory in
this order:

```
1. myapp / architecture       → project rules take precedence
2. myapp / anti-patterns      → confirmed local mistakes
3. react / patterns           → shared React knowledge fills in the rest
```

Merging rules:

- Project entries override shared entries **only when explicitly marked as a
  local override** (`label: local-override`). An undocumented contradiction
  is a potential inconsistency, not a silent override — surface it for human
  review.
- Within a wing/room, `invariant` entries apply first, then `decision`,
  `pattern`, then `note`. See
  [docs/retrieval.md](../docs/retrieval.md).
- Entries with `status: deprecated` are **not** surfaced unless explicitly
  requested.

### Fallback chain (graceful degradation)

If the memory backend is unavailable, the agent must not block. It falls back
to locally available sources, in order:

```
memory backend → docs/architecture.md → README.md → workspace search
```

Memory unavailability is never a reason to halt a task. See
[docs/architecture.md § Graceful Degradation](../docs/architecture.md#9-graceful-degradation).

---

## 4. Optional: Operational Rooms

When a project accumulates meaningful runtime behavior, add operational rooms —
but only once there is enough real content to justify them. Do not preallocate
empty rooms.

```
myapp/
  incident-log/    ← specific incidents that actually happened
  debugging/       ← repeatable diagnosis commands and investigation patterns
  observability/   ← logging, metrics, traces, and correlation guidance
  failure-modes/   ← generalized failure classes and recovery strategies
```

Keep room boundaries explicit so operational knowledge stays retrievable:

| Room            | Holds                                                           |
|-----------------|-----------------------------------------------------------------|
| `incident-log`  | One concrete incident: symptoms, root cause, fix, prevention.   |
| `failure-modes` | A generalized known risk for a component or subsystem.          |
| `debugging`     | How to inspect and diagnose (commands, dashboards, workflows).  |
| `observability` | What signals must exist so diagnosis is possible in the future. |

This separation is what keeps "we once saw this bug" from crowding out
"this is how you look for bugs of this class in general."

---

## 5. Deduplication Before Writing

Before adding a new entry, check for existing coverage **in the same wing and
room**. Scoping the comparison is what keeps the signal meaningful.

Suppose you want to add to `myapp/architecture`:

> "All external API calls must be wrapped in the repository layer."

Against the existing entry (see [§ 2.1](#21-architecture-rule--project-wing))
— "Data fetching must go through `src/repositories/`" — the new proposal is
essentially a restatement. The correct move is:

- **Enrich** the existing entry to make the scope explicit ("data fetching and
  other external API calls"), rather than creating a near-duplicate.
- **Preserve the original rule** and its rationale when enriching. Extend,
  don't rewrite.
- **Keep types separate.** If the proposed entry had instead been an
  `anti-pattern` about the same area, it would remain a separate entry even
  though the content overlaps, because its purpose (warning) is different
  from the architecture rule (constraint).

Similarity is guidance, not a gate. The primary question is always:

> *Does this entry add something genuinely distinct?*

If yes, create it. If not, enrich what exists.

Threshold ranges and the full policy are in
[docs/deduplication.md](../docs/deduplication.md).

---

## 6. What This Model Demonstrates

- **Scope separation is the first design decision.** Two wings is enough for
  most small projects — one project, one shared.
- **Rooms follow content, not anticipation.** Start with `architecture` and
  `anti-patterns`; add operational rooms only when real content appears.
- **Retrieval order is explicit.** Agents know where to look first, what
  overrides what, and what to do when memory is unavailable.
- **Deduplication is scoped.** Similarity is checked within the target wing
  and room, with type-aware exceptions.
- **Every entry has a reason to be where it is.** If you cannot explain why an
  entry is in its wing, it is probably in the wrong one.
