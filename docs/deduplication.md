# Semantic Deduplication and Memory Drift Control

Memory becomes noisy in two common ways: the same idea gets recorded more than once, and old entries stop matching reality.

---

## 1. Why deduplication matters

Deduplication is not just cleanup. It keeps memory trustworthy.

Duplicate or overlapping entries create:

- ambiguity during retrieval
- wasted context space
- drift between copies
- higher maintenance cost

The goal is not fewer entries for the sake of neatness. The goal is better signal per entry.

---

## 2. What counts as duplication

Duplication is not limited to copy-paste.

| Form | Description |
|------|-------------|
| **Lexical duplicate** | Same idea, almost the same wording |
| **Semantic duplicate** | Same rule or fact expressed differently |
| **Partial overlap** | One entry is mostly a subset of another |
| **Scattered accumulation** | The same guidance appears in several rooms |
| **Re-recorded decision** | A decision is written again instead of updated |

Example:

- "Nodes must not block callbacks"
- "Blocking inside a ROS 2 callback is not permitted"

These should usually be treated as one invariant, not two.

---

## 3. Compare within the right scope

Start with the smallest relevant scope.

### Start with the room

Most duplicates appear inside the same room.

### Expand to the wing when needed

If the same topic appears in neighboring rooms, compare there next.

### Treat project and shared wings separately

- **Project wings** hold local knowledge
- **Shared wings** hold reusable knowledge

If a project wing duplicates a shared rule, both copies need maintenance and will drift over time.

**Rule:** if a project entry duplicates a shared entry, deprecate the project copy and reference the shared one unless the project entry is a documented override.

---

## 4. Suggested similarity thresholds

If you use semantic similarity tooling, treat it as judgment support, not automation.

| Similarity score | Interpretation | Default action |
|------------------|----------------|----------------|
| **>= 0.86** | Near-duplicate | Enrich the existing entry |
| **0.55 - 0.85** | Related or possibly overlapping | Review manually |
| **< 0.55** | Likely distinct | A new entry may be justified |

Boundary cases still need human review.

---

## 5. Type-aware exceptions

Similar content does not always mean duplicate memory.

The same topic may legitimately appear as:

- an `invariant`
- a `decision`
- a `pattern`
- a `note`

Before merging two similar entries, confirm that they are the same type and serving the same role.

---

## 6. Enrich versus create

The default move is usually to **enrich** the existing entry.

### Enrich when

- the new content adds detail or rationale to the same rule
- the new content clarifies a boundary case
- the new content updates an incomplete entry

### Create when

- the rule or fact is genuinely distinct
- the content belongs in a different room or wing
- the existing entry is `deprecated` and the new content is a clean replacement

When enriching:

- keep the original `created_at`
- update `verified_at` if appropriate
- preserve the original rule

---

## 7. Preserve original rules when enriching

Enrichment should add clarity, not weaken the original meaning.

- do not weaken an invariant without approval
- do not silently widen scope
- do not merge types just to reduce entry count
- do not erase superseded meaning invisibly

If the rule changes in substance, that is usually a supersession event, not a simple enrichment.

---

## 8. Handling obsolete entries

An entry becomes obsolete when:

- the system it describes no longer exists
- the constraint has been removed
- the decision has been replaced

Do not delete obsolete entries silently. Keep them with `status: deprecated` and explain what replaced them or why they no longer apply.

### Deprecation workflow

1. Change status from `active` or `under_review` to `deprecated`
2. Add a `deprecated_at` date if your backend supports it
3. Explain what changed and what replaced the entry, if anything
4. Keep the deprecated entry out of default retrieval

If accuracy is uncertain, use `under_review` until someone confirms it.

---

## 9. Practical workflow before writing memory

Before creating a new entry:

1. identify the target wing and room
2. search that room for overlap
3. check nearby rooms in the same wing
4. check the shared wing if relevant
5. decide whether to enrich or create
6. confirm type and metadata
7. start new entries as `status: draft`

---

## 10. Common mistakes

**Writing first, searching second.**

**Comparing only titles.**

**Merging across types.**

**Updating one copy and forgetting the others.**

**Deleting instead of deprecating.**

**Treating thresholds as automation rules.**

---

## Checklist: before writing a new entry

```text
[ ] Wing and room identified
[ ] Room searched for existing entries on the same topic
[ ] Nearby rooms checked
[ ] Shared wing checked when relevant
[ ] Enrich-versus-create decision made
[ ] Type confirmed
[ ] Metadata complete
[ ] New entry starts as draft
[ ] No duplicate shared/project copy introduced
```