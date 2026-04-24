# Semantic Deduplication and Memory Drift Control

Memory accumulates redundancy and drift over time. This guide explains how to detect, classify, and resolve both — before writing new entries and as part of ongoing maintenance.

Deduplication is not a cleanup task. It is a design discipline. An agent that retrieves two conflicting entries covering the same constraint cannot determine which is authoritative. The result is inconsistent behavior that looks like a model problem but is a memory problem.

---

## 1. Why Deduplication Matters

Every duplicate entry increases the cost of retrieval and reduces its reliability:

- **Ambiguity during retrieval.** When two entries address the same concept with different wording or contradictory content, the agent has no principled way to resolve the conflict. It may follow the wrong one, blend both incorrectly, or surface the contradiction to the user without resolution.
- **Context window pressure.** Retrieving near-duplicate entries wastes space that should go to distinct, complementary context.
- **Silent drift amplification.** When the same fact exists in three entries, updating one without updating the others creates a three-way split in authority. One entry gets corrected; two continue to mislead.
- **Maintenance cost multiplier.** Every time a constraint changes, the number of entries requiring update scales with the degree of duplication.

A memory system with well-controlled deduplication retrieves fewer entries, with higher signal per entry, and requires less maintenance per change.

---

## 2. What Counts as Duplication

Duplication is not only exact repetition. It occurs in several forms:

| Form | Description |
|------|-------------|
| **Lexical duplicate** | Same content, same type, different wording |
| **Semantic duplicate** | Different wording, but the constraint or fact is the same |
| **Partial overlap** | One entry is a strict subset of another |
| **Scattered accumulation** | The same pattern is recorded independently in multiple rooms without cross-reference |
| **Re-recorded decisions** | A decision that was already active was written again after being forgotten |

Semantic duplication is the hardest to catch. Two entries like "Nodes must not block callbacks" and "Blocking inside a ROS 2 callback is not permitted" express the same invariant and should be merged — even if their phrasing differs entirely.

What does **not** count as duplication is addressed in section 5.

---

## 3. Wing and Room Scoped Comparison

Deduplication should be performed within a defined scope. Comparing all entries globally is impractical and can lead to false positives — entries that share surface similarity but operate in different domains.

### Start within the room

The most common source of duplication is within a single room. Before writing a new entry, compare against all existing entries in the same room.

### Expand to the wing when needed

Cross-room duplication within a wing occurs when the same pattern is independently discovered or recorded in multiple sub-topics. After a batch of new entries, scan across rooms within the active wing.

### Handle project and shared wings separately

Wings typically fall into two categories:

- **Project wings** — Scoped to a specific project (e.g., `lifecore_ros2`, `myapp`). Entries here reflect decisions made for that project.
- **Shared wings** — Reusable conventions applicable across projects (e.g., `ros2`, `react`).

Duplication *across* these two categories is particularly risky. If a constraint that belongs in a shared wing is also recorded in a project wing, any update to one will not automatically propagate to the other. The entries drift apart. When an agent loads both wings, it may apply the wrong version for the context it is in.

**Rule:** If an entry in a project wing duplicates a constraint already in a shared wing, deprecate the project-wing copy and reference the shared-wing entry. Only keep a project-wing entry if it genuinely overrides or qualifies the shared constraint.

---

## 4. Suggested Similarity Thresholds

When using semantic similarity tools to compare candidate entries, the following ranges provide working guidance for how to act on results. These are **thresholds for judgment, not hard enforcement rules.** Context, entry type, and scope all affect the correct interpretation.

| Similarity score | Interpretation | Default action |
|-----------------|---------------|----------------|
| **≥ 0.86** | Near-duplicate | Enrich the existing entry rather than creating a new one |
| **0.55 – 0.85** | Related, possibly overlapping | Review manually; determine whether entries address distinct aspects |
| **< 0.55** | Likely distinct | Create a new entry, but verify scope and type before proceeding |

A score at the boundary (e.g., 0.84 or 0.87) should be reviewed by a human before acting. Automated tooling that applies these thresholds should flag boundary cases rather than acting on them unilaterally.

When no similarity tooling is available, the comparison is done by reading. The same thresholds apply conceptually — ask whether the entries would tell an agent the same thing in context.

---

## 5. Type-Aware Exceptions

The same underlying content expressed in two different entry types does not always constitute duplication.

Consider a constraint like "All API endpoints must be versioned":

- An `invariant` entry expresses this as a non-negotiable rule that must never be violated.
- A `decision` entry records the rationale and history behind the choice.
- A `pattern` entry describes how to implement versioning correctly.
- A `note` might document a specific exception or edge case.

All four can coexist. They serve different retrieval purposes. Merging them would collapse the retrieval hierarchy and lose the semantic weight attached to each type.

**Type-aware check:** Before merging two entries that appear to cover the same topic, verify they are the same type. If they are not, ask whether each is performing a distinct role in the retrieval hierarchy. If yes, they should remain separate. If one is an informal copy of the other at the wrong type level, deprecate the weaker one.

---

## 6. Enrich vs. Create

When comparison reveals that a closely related entry already exists, the default action is to **enrich the existing entry** rather than creating a new one.

**Enrich when:**
- The new content adds detail, context, or a rationale to an existing entry
- The new content clarifies a boundary case not covered explicitly
- The new content updates a fact that was previously incomplete

**Create a new entry when:**
- The new content addresses a genuinely distinct constraint or pattern
- The new content belongs in a different room or wing from the existing entry
- The existing entry is `deprecated` and the new content represents a clean replacement

When enriching, preserve the original `created_at` date. Update `verified_at` to reflect the revision. Note what was changed in a brief inline comment if the content shift is significant. Do not silently rewrite an entry in a way that obscures its history.

---

## 7. Preserving Original Rules

Enrichment does not mean overwriting. When updating an existing entry:

- **Do not weaken an invariant** without human approval. Softening "must not" to "should avoid" is a substantive modification, not a clarification — treat it as a supersession, subject to rules R5, R13, and R14 in `principles/rules.md`.
- **Do not expand the scope** of an entry beyond its original intent. If the original entry applies to one room and the new content applies more broadly, draft a separate entry rather than silently widening the existing one.
- **Do not merge types.** If an invariant and a decision cover related content, enriching one with content from the other conflates their roles in the retrieval hierarchy.
- **Reference supersession explicitly.** If the enrichment effectively replaces a key claim in the original, mark the old version's claim as superseded within the entry rather than silently erasing it.

These constraints apply to agent-authored enrichments and human-authored ones equally.

---

## 8. Handling Obsolete Entries

An entry becomes obsolete when the system it describes no longer exists, the constraint it encodes has been removed, or the decision it records has been reversed.

Obsolete entries are **not deleted.** They are **deprecated with a reference to what replaced them or why they no longer apply.** This is rule R14 in `principles/rules.md` and is non-negotiable.

### Deprecation workflow

1. Change status from `active` (or `under_review`) to `deprecated`.
2. Add a `deprecated_at` date.
3. Add a note in the entry body: what changed, and what the replacement entry is (if any).
4. If no replacement exists, note the reason for deprecation explicitly.

Deprecated entries are excluded from standard retrieval (see `docs/retrieval.md`). They remain available for audit and traceability.

### Drift as a form of functional obsolescence

An entry that has not drifted to `deprecated` but whose content no longer reflects the system is functionally obsolete. This is more dangerous than an explicitly deprecated entry because it participates in standard retrieval with full authority.

Mark any entry whose accuracy you cannot confirm as `under_review`. Do not let uncertainty linger silently in an `active` entry.

---

## 9. Practical Workflow Before Writing Memory

Before creating any new memory entry, run this sequence:

1. **Identify the target wing and room.** Confirm which wing and room the new entry belongs in. Do not default to a project wing if the content belongs in a shared wing.

2. **Search within that room first.** Look for entries covering the same topic. Read them, not just their titles.

3. **Check adjacent rooms within the wing.** Scattered accumulation often spans rooms within the same wing.

4. **Check the corresponding shared wing** (if a shared wing exists). If the same constraint is already in a shared wing, do not duplicate it in a project wing unless a genuine override is needed.

5. **Apply the similarity judgment.** Score or assess conceptual overlap. Use the thresholds in section 4 as a starting frame, adjusted for type and scope.

6. **Decide: enrich or create.** If enriching, follow the rules in sections 6 and 7. If creating, confirm the type is correct and metadata is complete.

7. **Draft, do not activate.** New entries start with `status: draft`. Activation requires human approval, especially for `invariant` and `decision` entries (rule R5).

---

## 10. Common Mistakes

**Writing first, checking second.** The most common source of duplication is not malice — it is skipping the pre-write check. Once an entry is `active`, removing it requires explicit deprecation and creates a trace. Prevention is cheaper than remediation.

**Comparing titles instead of content.** Two entries with different slugs can be near-duplicates. Similarity comparison must be done on entry body content, not identifiers.

**Merging types to reduce count.** Combining an invariant and a decision into a single entry to "reduce clutter" destroys the retrieval hierarchy. The agent can no longer distinguish a hard constraint from a considered choice. Keep types separate.

**Updating one copy, forgetting others.** When the same constraint exists in multiple entries, a targeted update to one leaves the others stale. Before updating any entry, check for semantic duplicates that may need the same update.

**Soft-deleting instead of deprecating.** Removing an entry without a deprecation trace eliminates the record of why the constraint existed. Future agents or engineers have no way to understand why the entry is missing or whether its removal was intentional.

**Assuming cross-wing entries are independent.** A project-wing entry and a shared-wing entry covering the same constraint are not independent. They are competing authorities. If both exist, one should explicitly reference or supersede the other.

**Treating the thresholds as automation triggers.** Similarity scores are signals for human judgment, not commands for automated action. A score of 0.87 means "review carefully before enriching," not "merge automatically."

---

## Checklist: Before Writing a New Entry

```
[ ] Wing and room identified
[ ] Room searched for existing entries on the same topic
[ ] Adjacent rooms within the wing checked
[ ] Corresponding shared wing checked (if applicable)
[ ] Similarity assessed — enrich or create decision made
[ ] If enriching: original rules and type preserved
[ ] If creating: type confirmed, metadata complete (type, status: draft, created_at)
[ ] No project-wing duplicate of a shared-wing entry introduced
[ ] Entry will not be marked active without human approval
```

---

## Summary

| Process | Trigger | Outcome |
|---------|---------|---------|
| Pre-write check | Before every new entry | Duplication caught before it enters the system |
| Room-level deduplication | After a batch of entries; monthly | Redundant entries enriched or deprecated |
| Cross-wing audit | Before a new agent session; on significant change | Project vs. shared wing conflicts resolved |
| Drift check | Regular schedule; pre-session | Stale entries flagged as `under_review` |
| Supersession | Decision or invariant changes | Old entry deprecated with back-reference; new entry created |
