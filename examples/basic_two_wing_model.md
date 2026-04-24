# Example: Basic Two-Wing Memory Model

This example shows the smallest useful structure for long-term memory: one project wing and one shared wing.

The goal is simple: separate what is local to one codebase from what should stay reusable across projects.

---

## 1. Setup: two wings, three rooms

```text
myapp/              <- project wing
  architecture/     <- project-specific design rules
  anti-patterns/    <- confirmed mistakes observed in this project

react/              <- shared wing
  conventions/      <- reusable React implementation guidance
```

Why two wings?

- the **project wing** holds rules that only make sense for `myapp`
- the **shared wing** holds knowledge reusable across multiple React projects

That separation prevents the Project Island problem, where framework guidance gets copied into every project and slowly drifts.

---

## 2. Sample entries

### 2.1 Architecture rule — project wing

**Wing:** `myapp` · **Room:** `architecture`

```text
type: invariant
label: architecture-rule
status: active
created_at: 2026-03-10

All data fetching must go through the repository layer (`src/repositories/`).
Components must not call the API client directly.

Rationale: keeps components testable and isolates network concerns.
```

Why here: this rule is specific to `myapp`.

### 2.2 Anti-pattern — project wing

**Wing:** `myapp` · **Room:** `anti-patterns`

```text
type: invariant
label: anti-pattern
status: active
created_at: 2026-03-18

Do not store derived state in Redux when it can be computed from existing state.
This caused a stale cart total after a coupon was applied.
Compute totals in selectors instead.

Rationale: derived state drifts from its inputs and creates stale-read bugs.
```

Why here: this is a project-specific mistake tied to a real incident.

### 2.3 Shared knowledge — shared wing

**Wing:** `react` · **Room:** `conventions`

```text
type: pattern
label: reusable-pattern
status: active
created_at: 2026-01-20

React Context is a good fit for low-frequency global state such as theme,
locale, or auth. Do not use Context for high-frequency updates like form
fields or animation state. Use local state or a dedicated store instead.
```

Why here: this guidance is reusable across React projects.

---

## 3. Retrieval order

Before changing a data-fetching component, gather context in this order:

```text
1. myapp / architecture
2. myapp / anti-patterns
3. react / conventions
```

Rules for merging:

- project entries only override shared entries when the local override is explicit
- within any room, apply `invariant` before `decision`, then `pattern`, then `note`
- entries with `status: deprecated` stay out of default retrieval

### Fallback chain

If the memory backend is unavailable:

```text
memory backend -> docs/architecture.md -> README.md -> workspace search
```

---

## 4. Optional operational rooms

When a project grows enough operational history:

```text
myapp/
  incident-log/
  debugging/
  observability/
  failure-modes/
```

| Room | Holds |
|------|-------|
| `incident-log` | One concrete incident |
| `failure-modes` | A generalized recurring risk |
| `debugging` | Investigation workflows |
| `observability` | The signals needed to diagnose future issues |

---

## 5. Deduplication before writing

Suppose you want to add this to `myapp/architecture`:

> All external API calls must be wrapped in the repository layer.

Compared with the existing architecture rule above, this is basically a restatement.

The better move is to enrich the existing entry instead of creating a duplicate.

---

## 6. What this model demonstrates

- scope separation comes first
- rooms should follow real content
- retrieval order should be explicit
- deduplication should be scoped early
- every entry should have a clear reason to exist where it exists