# Template: Memory Entry

This file provides standard templates for common durable memory entries.

Copy the block you need, fill it in, and store it in the right wing and room.

---

## Conventions used throughout

- the **primary type** is one of `invariant`, `decision`, `pattern`, or `note`
- the **status** is one of `draft`, `active`, `under_review`, or `deprecated`
- a **label** can add precision for humans, but it does not replace the primary type
- agents may draft entries, but activation requires human approval

Entry IDs should be lowercase, hyphen-separated, and descriptive enough to recognize quickly.

---

## Shared field reference

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `invariant`, `decision`, `pattern`, or `note` |
| `label` | no | Human-readable secondary tag |
| `status` | yes | `draft`, `active`, `under_review`, or `deprecated` |
| `created_at` | yes | ISO 8601 date when the entry was first written |
| `verified_at` | no | ISO 8601 date when the entry was last confirmed accurate |
| `version` | no | Integer version if the rule itself evolves |
| `supersedes` | no | ID of an older entry this one replaces |
| `superseded_by` | no | ID of a newer entry that replaces this one |

---

## 1. architecture-rule

High-level structural guidance.

**Typical location:** `<project_wing>/architecture`

```text
## <entry-id>

type: invariant          # or: decision
label: architecture-rule
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

<State the rule directly. Use must or must not when appropriate.>

Rationale: <why the rule exists and what breaks if it is ignored>

[see: <wing>/<room>/<entry-id>]   # optional
```

---

## 2. component-contract

A contract between components.

**Typical location:** `<project_wing>/components` or `<project_wing>/contracts`

```text
## <entry-id>

type: decision
label: component-contract
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Component: <name or path>
Consumers: <callers or "any">

Contract:
- Inputs: <preconditions and validation expectations>
- Outputs: <postconditions and guarantees>
- Errors: <how failures are signaled>
- Threading: <threading or execution assumptions>

Rationale: <why this contract shape matters>
```

---

## 3. anti-pattern

A specific mistake that should not recur.

**Typical location:** `<project_wing>/anti-patterns`

```text
## <entry-id>

type: invariant
label: anti-pattern
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Do not: <precise description of the forbidden pattern>

Observed impact: <bug, failure, or incident that motivated this entry>

Do instead: <the correct alternative>

Rationale: <why the forbidden pattern fails>
```

---

## 4. code-convention

A consistency rule that matters, but does not rise to the level of an architectural constraint.

**Typical location:** `<project_wing>/conventions` or `<shared_wing>/conventions`

```text
## <entry-id>

type: pattern
label: code-convention
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Applies to: <file pattern, language, module, or component scope>

Convention: <what to do>

When to apply: <scope>
When not to apply: <exceptions>

Rationale: <why this convention exists>
```

---

## 5. migration-note

A record of a breaking change, deprecation path, or upgrade strategy.

**Typical location:** `<project_wing>/migration-notes`

```text
## <entry-id>

type: note               # or: decision if the migration path is itself a choice
label: migration-note
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional
version: 1               # optional

From: <old version, API, or schema>
To:   <new version, API, or schema>

Migration steps:
1. <step>
2. <step>
3. <step>

Compatibility window: <timing>
Rollback plan: <how to revert, or "not supported">

Rationale: <why the migration exists>

supersedes: <entry-id>   # optional
```

---

## 6. incident-log

A record of one concrete incident.

**Typical location:** `<project_wing>/incident-log`

```text
## <entry-id>

type: note
label: incident-log
status: draft
created_at: YYYY-MM-DD

Summary: <one-sentence description>

Symptoms:
- <observable symptom>
- <observable symptom>

Timeline:
- <timestamp> — <event>
- <timestamp> — <event>

Root cause: <precise cause>

Fix: <what resolved the issue>

Prevention:
- <test, alert, invariant, anti-pattern, or other follow-up>
```

Historical records may eventually become `active`, but agent-authored entries should still start as `draft`.

---

## 7. failure-mode

A generalized class of failure: what triggers it, what it looks like, and how to recover.

**Typical location:** `<project_wing>/failure-modes`

```text
## <entry-id>

type: note               # or: decision if recovery policy is prescriptive
label: failure-mode
status: draft
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD  # optional

Component: <name or subsystem>
Failure class: <short descriptor>

Triggers:
- <condition>
- <condition>

Symptoms:
- <what the failure looks like>

Detection: <how it is detected>

Recovery strategy:
- <step or policy>

Preventive measures:
- <test, invariant, observability rule, or other control>
```

---

## Entry writing guidelines

- one concept per entry
- write direct statements
- avoid unclear pronouns
- keep entries short enough to review quickly
- `verified_at` tracks review freshness, not just edits
- when replacing an entry, mark the old one `deprecated` and fill in `superseded_by`