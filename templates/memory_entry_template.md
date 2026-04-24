# Template: Memory Entry

Standard templates for the most common kinds of memory entries. Copy the
block you need, fill it in, and store it in the appropriate wing and room.

Conventions used throughout:

- The **primary type** (`invariant`, `decision`, `pattern`, `note`,
  `deprecated`) drives retrieval and weighting. See
  [docs/architecture.md § Entry Types](../docs/architecture.md#entry-types).
- The **label** is a finer-grained tag that clarifies intent for humans.
  Labels do not replace the primary type.
- Metadata fields follow the repository standard:
  `type`, `status`, `created_at`, and optionally `verified_at`, `version`,
  `supersedes`, `superseded_by`.
- Status vocabulary: `draft`, `active`, `under_review`, `deprecated`.
- Agents may only create entries with `status: draft`. Activation requires
  human approval (see [principles/rules.md § R5](../principles/rules.md)).

Entry IDs should be lowercase, hyphen-separated, and descriptive enough to be
recognizable without reading the full content (e.g. `no-blocking-callbacks`,
`repository-layer-required`).

---

## Shared Field Reference

| Field           | Required | Description                                                          |
|-----------------|----------|----------------------------------------------------------------------|
| `type`          | yes      | One of: `invariant`, `decision`, `pattern`, `note`, `deprecated`.    |
| `label`         | no       | Finer-grained intent tag (see per-template examples below).          |
| `status`        | yes      | One of: `draft`, `active`, `under_review`, `deprecated`.             |
| `created_at`    | yes      | ISO 8601 date when the entry was first written.                      |
| `verified_at`   | no       | ISO 8601 date when the entry was last confirmed accurate.            |
| `version`       | no       | Integer, incremented when the rule itself changes.                   |
| `supersedes`    | no       | ID of the entry this one replaces.                                   |
| `superseded_by` | no       | ID of the entry that replaces this one (use when deprecating).       |

---

## 1. architecture-rule

A high-level, project-wide constraint on how the system is structured. Most
architecture rules are modeled as `invariant` (must never be violated) or
`decision` (deliberate structural choice with rationale).

**Typical location:** `<project_wing>/architecture`.

```
## <entry-id>

type: invariant          # or: decision
label: architecture-rule
status: draft            # activate only after human approval
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

<State the rule as a single direct constraint. Use "must" / "must not".>

Rationale: <why this rule exists; what breaks if it is violated>

[see: <wing>/<room>/<entry-id>]   # optional, for related or shared entries
```

---

## 2. component-contract

A guarantee or obligation between two components: inputs, outputs, ordering,
error behavior, threading model. Component contracts are almost always
`decision` entries — they encode a deliberate interface choice.

**Typical location:** `<project_wing>/components` or `<project_wing>/contracts`.

```
## <entry-id>

type: decision
label: component-contract
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Component: <name or path>
Consumers: <list of callers or "any">

Contract:
- Inputs:  <preconditions, types, validation expectations>
- Outputs: <postconditions, types, ordering guarantees>
- Errors:  <how failures are signaled; what callers must handle>
- Threading: <single-threaded / thread-safe / lock requirements>

Rationale: <why this contract shape; what would break if it changed>

[see: <wing>/<room>/<entry-id>]   # optional
```

---

## 3. anti-pattern

A specific mistake that has been observed (or is highly likely) and must be
avoided. Anti-patterns are modeled as `invariant` entries (must not happen),
tagged with a label so they can be surfaced distinctly from positive rules.

**Typical location:** `<project_wing>/anti-patterns`.

```
## <entry-id>

type: invariant
label: anti-pattern
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Do not: <precise description of the forbidden pattern>

Observed impact: <incident, bug, or concrete failure that motivated this entry>

Do instead: <the correct alternative, stated directly>

Rationale: <why the forbidden pattern fails; root cause>
```

---

## 4. code-convention

A style, naming, or structural preference that should be applied consistently
but does not rise to the level of an architectural constraint. Code
conventions are `pattern` entries.

**Typical location:** `<project_wing>/conventions` or `<shared_wing>/conventions`.

```
## <entry-id>

type: pattern
label: code-convention
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Applies to: <file pattern, language, module, or component scope>

Convention: <what to do, stated positively>

When to apply: <the scope in which this applies>
When not to apply: <explicit exceptions, if any>

Rationale: <why this convention; what inconsistency it prevents>
```

---

## 5. migration-note

A record of a breaking change, deprecation, or upgrade path. Migration notes
are usually `note` entries (informational) but can be `decision` entries when
the migration path itself is a deliberate choice.

**Typical location:** `<project_wing>/migration-notes`.

```
## <entry-id>

type: note               # or: decision, when the migration path is a choice
label: migration-note
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional
version: 1               # optional; bump when the migration plan changes

From: <old version, API, or schema>
To:   <new version, API, or schema>

Migration steps:
1. <step>
2. <step>
3. <step>

Compatibility window: <e.g. "90 days after release X.Y">
Rollback plan: <how to revert, or "not supported">

Rationale: <why this migration is needed; what it unlocks or prevents>

supersedes: <entry-id>   # optional, if this migration replaces a previous plan
```

---

## 6. incident-log

One concrete incident that actually happened: symptoms, diagnosis, root cause,
fix, and prevention. Incident-log entries are `note` entries — they are
historical records, not constraints. Constraints derived from an incident
belong in their own `invariant` or `anti-pattern` entry that references the
incident.

**Typical location:** `<project_wing>/incident-log`.

```
## <entry-id>

type: note
label: incident-log
status: active           # historical records are usually active immediately
created_at: YYYY-MM-DD   # date the incident occurred

Summary: <one-sentence description of the incident>

Symptoms:
- <observable symptom>
- <observable symptom>

Timeline:
- <timestamp> — <event>
- <timestamp> — <event>

Root cause: <precise cause, not just the trigger>

Fix: <what was changed to resolve the incident>

Prevention:
- <what was added: test, alert, invariant, anti-pattern entry, etc.>

[see: <wing>/anti-patterns/<entry-id>]  # optional, if an anti-pattern was created
[see: <wing>/failure-modes/<entry-id>]  # optional, if a failure mode was generalized
```

---

## 7. failure-mode

A generalized class of failure for a component or subsystem: what triggers it,
how it manifests, and the recovery strategy. Failure modes are `note` or
`decision` entries depending on whether they prescribe a recovery policy.

Unlike `incident-log` (one thing that happened), a failure mode generalizes
across incidents and describes a class of risk.

**Typical location:** `<project_wing>/failure-modes`.

```
## <entry-id>

type: note               # or: decision, when the recovery strategy is prescriptive
label: failure-mode
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Component: <name or subsystem>
Failure class: <short descriptor, e.g. "message backlog", "partial write">

Triggers:
- <condition that can cause this failure>
- <condition that can cause this failure>

Symptoms:
- <what the failure looks like from outside>

Detection: <how this failure is currently detected; what signal exposes it>

Recovery strategy:
- <steps or policy to recover>

Preventive measures:
- <tests, invariants, or observability that reduce probability>

[see: <wing>/incident-log/<entry-id>]   # optional, incidents that exemplify this class
[see: <wing>/observability/<entry-id>]  # optional, signals that detect this class
```

---

## Entry Writing Guidelines

- **One concept per entry.** If the entry needs two distinct headings to
  organize its content, it is probably two entries.
- **Entries are constraints, not documentation.** Write direct statements.
  Avoid narrative prose.
- **Avoid pronouns without referents.** "It must not do this" is not useful
  without a clear subject and action.
- **Keep entries reviewable in under two minutes.** If an entry is longer
  than that, split it.
- **`verified_at` tracks reviews, not edits.** A review that confirms "still
  correct, no change needed" still bumps `verified_at`.
- **Supersession is explicit.** When replacing an entry, set
  `status: deprecated` on the old one and populate `superseded_by`. Silent
  deletion is not permitted — see
  [principles/rules.md § R14](../principles/rules.md).
