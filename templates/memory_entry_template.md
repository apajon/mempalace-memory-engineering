# Template: Memory Entry

Use this template when creating a new memory entry. Fill in all required fields. Remove optional fields only if they genuinely do not apply.

Entries should be stored in the appropriate wing and room. The entry ID should be lowercase, hyphen-separated, and descriptive enough to be recognizable without reading the full content.

---

```markdown
## [entry-id]

type: [invariant | decision | pattern | note | deprecated]
status: [draft | active | under_review | deprecated]
created_at: YYYY-MM-DD
verified_at: YYYY-MM-DD
wing: [wing-name]
room: [wing-name/room-name]

<!-- Optional: reference to superseded entry -->
supersedes: [entry-id or "n/a"]

<!-- Optional: reference to entry that supersedes this one (use when status is deprecated) -->
superseded_by: [entry-id or "n/a"]

---

[Entry content here. Write in clear, direct language. Avoid jargon unless it is domain-specific
and expected to be understood by all agents and humans working in this wing.]

[For invariants: state the constraint and, if not obvious, the consequence of violation.]
[For decisions: state the decision and its rationale. Include what alternatives were considered
if that context is useful.]
[For patterns: state the approach and when to apply it. Include when not to apply it if
the pattern is frequently misapplied.]
[For notes: state the information. Notes do not require rationale but should be specific enough
to be useful.]
```

---

## Field Reference

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | One of: `invariant`, `decision`, `pattern`, `note`, `deprecated` |
| `status` | Yes | One of: `draft`, `active`, `under_review`, `deprecated` |
| `created_at` | Yes | ISO 8601 date when the entry was first written |
| `verified_at` | Yes | ISO 8601 date when the entry was last confirmed accurate |
| `wing` | Yes | The wing this entry belongs to |
| `room` | Yes | The full room path (e.g., `ros2/invariants`) |
| `supersedes` | No | ID of entry this one replaces, if applicable |
| `superseded_by` | No | ID of entry that replaces this one (used when deprecating) |

---

## Entry Writing Guidelines

- **One concept per entry.** If the entry requires two distinct headings, it is probably two entries.
- **Entries are not documentation.** They are constraints, decisions, and patterns. Do not write narrative prose. Write direct statements.
- **Avoid pronouns without clear referents.** "It should not do this" is not useful without a clear subject and action.
- **Entries should be reviewable by a human in under two minutes.** If an entry is longer than that, consider splitting it.
- **`verified_at` must be updated when the content is reviewed, not only when it is changed.** A review that confirms no change is still a review.
