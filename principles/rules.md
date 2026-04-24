# Operational Rules

These rules translate the design invariants into practical decisions an author makes while working with memory.

---

## 1. When to create a wing

Create a wing when you have a real scope boundary, not just a new topic.

Create a new wing when:

- the content belongs to a distinct project scope
- the content is reusable knowledge that will serve more than one project

Do not create a wing for:

- a subtopic inside an existing wing
- a single task or sprint
- one person's preferences

---

## 2. When to create a room

Create a room when an existing room has become too broad or when a new subject clearly does not fit any current room.

Good triggers:

- one room now covers two clearly different subjects
- you need to write an entry and no current room fits it well
- a recurring topic matches a standard room slug such as `incident-log` or `failure-modes`

Do not create rooms preemptively.

---

## 3. When to write an entry

Write an entry only when all of the following are true:

1. the content is non-obvious, reusable, and likely to matter again
2. you checked the target room first
3. you can express it as one durable rule or fact
4. the content has a clear type

---

## 4. When not to write memory

Do not persist:

- session-scoped debugging notes
- framework documentation copied from upstream sources
- one-off fixes with no likely recurrence
- speculative rules that are not yet validated
- personal preferences the project has not adopted
- anything you have not compared against existing memory first

---

## 5. When to enrich instead of creating

Enriching an existing entry is the default when the new content is really an extension of the same rule.

Prefer enrichment when:

- the same constraint already exists in the room
- the new content adds rationale, detail, or a boundary case
- the overlap is high enough that both entries would tell the agent almost the same thing

Create a new entry when:

- the new content is a genuinely separate rule
- it belongs in a different room or wing
- it serves a different type role

---

## 6. When to mark obsolete

Mark an entry `deprecated` when:

- it no longer matches the current system
- a replacement entry supersedes it
- the feature or component it described is gone

Use `under_review` when you suspect staleness but have not confirmed it yet.

When deprecating an entry:

1. set `status: deprecated`
2. reference the replacement if one exists
3. keep the old entry for traceability
4. exclude it from default retrieval

---

## 7. When to cross-reference

Cross-reference instead of copying whenever one entry depends on knowledge that already lives elsewhere.

Good uses:

- a project entry depends on a shared rule
- two rooms cover related parts of the same topic
- a decision explains the rationale behind an invariant

---

## 8. How to resolve conflicts

Two entries conflict when they cover the same topic and disagree on the rule.

Classify the conflict first:

- **Intentional override**
- **Undocumented divergence**
- **Authoring failure**
- **Governance failure**

Then act accordingly:

- verify and keep intentional overrides
- flag undocumented divergence for human review
- deprecate the stale entry in an authoring failure
- move conflicting invariants to `under_review` until a human resolves them

Never resolve contradictions by silently merging them.

---

## Quick reference

| Question | Rule |
|----------|------|
| New scope boundary? | Create a wing |
| Existing room too broad? | Create a room |
| Durable, reusable content? | Write an entry |
| Session-only or obvious content? | Do not write memory |
| Related entry already exists? | Enrich instead of creating |
| Entry no longer matches reality? | Mark obsolete |
| One entry depends on another? | Cross-reference |
| Two entries disagree? | Resolve the conflict explicitly |