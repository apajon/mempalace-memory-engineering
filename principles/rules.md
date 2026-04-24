# Operational Rules

These rules translate the design invariants in
[principles/invariants.md](invariants.md) into concrete decisions an author — human or
agent — makes while working with memory. Each rule answers one question.

Structural terminology (wings, rooms, entries, types, status) is defined in
[docs/architecture.md](../docs/architecture.md). Deduplication thresholds and scope are
defined in [docs/deduplication.md](../docs/deduplication.md).

These rules are practical guidance distilled from the methodology. They are not validated
best practices from large-scale use. Treat them as a working baseline, not a standard.

---

## 1. When to create a wing

Create a wing when you have a distinct **scoping boundary**, not when you have a new topic.

A new wing is justified when:

- The content is scoped to a project that does not share knowledge with existing project
  wings, **or**
- The content is a reusable body of knowledge (a framework, a platform, a language) that
  will be consumed by more than one project.

Do **not** create a wing for:

- A subtopic inside an existing wing — that is a room.
- A single task or sprint.
- A per-agent or per-user preference scope.

**Check.** Before creating wing `X`, ask: *Will retrieval in another wing need to cross
into `X`? If yes, is that a cross-wing reference, or should this content live in the
existing wing?* If the content is only ever retrieved in one existing wing, it belongs in
that wing.

See [docs/architecture.md §5 Scope Separation](../docs/architecture.md#5-scope-separation).

---

## 2. When to create a room

Create a room when an existing room is doing too much and retrieval inside it has become
noisy.

Triggers:

- The room contains entries about two clearly distinct subjects (e.g., component contracts
  and debugging workflows).
- You are about to write an entry and no existing room fits its subject.
- A standard room slug from the architecture doc (e.g., `anti-patterns`,
  `incident-log`, `failure-modes`) applies and the topic keeps recurring.

Do **not** create a room:

- Preemptively. Empty rooms are noise.
- For a single entry that fits an existing room with minor stretching — enrich the fit
  instead.
- To mirror a code-tree layout. Rooms are retrieval categories, not folders.

**Starting point.** Begin each wing with `architecture` and `anti-patterns`. Add more when
content justifies them. See
[docs/architecture.md §4 Core Model → Rooms](../docs/architecture.md#rooms).

---

## 3. When to write an entry

Write an entry when all of the following hold:

1. The content passes the durability test from invariant **I5**: non-obvious, reusable,
   and likely to matter again.
2. You have searched the target room (and target wing, for shared concerns) and confirmed
   no existing entry already covers it — see rule **§5 When to enrich instead of creating**.
3. You can state it as a single rule or fact (invariant **I8**). If you cannot, split it.
4. The content has a clear type: `invariant`, `decision`, `pattern`, or `note`. If you
   cannot assign a type, the content is probably not ready to persist.

**Example — worth writing:**

> Topic components must gate message publication on node activation state. Subscribers
> must check node state before forwarding messages.
>
> *Non-obvious, reusable across every topic component in the project, prevents a known
> class of bug.*

**Example — not worth writing:**

> The `configure()` method sets up the node.
>
> *Obvious from the framework docs. Not durable memory; it is restated documentation.*

---

## 4. When not to write memory

Do not write an entry for any of the following:

- **Session-scoped context.** Debugging notes for the current task, intermediate results,
  conversation summaries. These expire with the session.
- **Framework or library documentation.** If it is in the upstream docs, link to the docs;
  do not restate them.
- **One-off fixes.** A single bug patched once, with no class of recurrence, is a commit
  message, not an entry.
- **Speculative rules.** A pattern noticed once and not yet validated. Wait until it has
  recurred or been decided.
- **Personal preferences without a project decision.** Style preferences become entries
  only when the project has adopted them.
- **Content you have not read the existing memory to compare against** — writing without
  reading is ruled out by invariant **I4**.

**Heuristic.** *If this entry were retrieved six months from now on a different task,
would the agent still benefit from it?* If no, do not write it.

---

## 5. When to enrich instead of creating

Enriching an existing entry is the default when related content already exists. Prefer
enrichment when:

- An existing entry in the same room addresses the same constraint, even if phrased
  differently. This is the semantic-duplicate case from
  [docs/deduplication.md §2](../docs/deduplication.md#2-what-counts-as-duplication).
- The new content is a **refinement, qualification, or additional rationale** for an
  existing rule — not a new rule.
- Similarity against the existing entry is in the high range (see
  [deduplication thresholds](../docs/deduplication.md#4-suggested-similarity-thresholds)):
  near-duplicate → enrich; related but distinct → review manually.

Create a new entry instead when:

- The rule is of a different **type** than the existing entry (e.g., an `invariant` where
  only a `pattern` exists). Type-aware exception from
  [docs/deduplication.md §5](../docs/deduplication.md#5-type-aware-exceptions).
- The new content genuinely adds a separate atomic rule (invariant **I8**).

**When enriching, preserve the original rule.** Extend it; do not rewrite it from scratch.
Update `verified_at` if present. Leave `created_at` alone.

---

## 6. When to mark obsolete

Mark an entry as `deprecated` when any of the following is true:

- The rule no longer matches the current system (drift confirmed against code or
  behavior).
- A replacement entry has been written that supersedes it.
- The rule was scoped to a feature or component that has been removed.

Procedure:

1. Set `status: deprecated`.
2. Add a reference to the replacement entry, if one exists:
   `superseded_by: wing/room/entry-id`.
3. Do **not** delete the old entry. Traceability requires the deprecated record to remain.
4. The deprecated entry must stop appearing in default retrieval. It may still be fetched
   on explicit request.

Mark `status: under_review` (not `deprecated`) when you suspect an entry is stale but have
not confirmed it. `under_review` signals uncertainty; `deprecated` signals a confirmed
replacement or removal.

This follows invariant **I7** and the supersession semantics in
[docs/architecture.md §8 Persistence Rules](../docs/architecture.md#8-persistence-rules).

---

## 7. When to cross-reference

Cross-reference instead of copying whenever a project entry depends on a rule that lives
in a shared wing.

**Pattern:**

```
[see: ros2/lifecycle] — this component extends the lifecycle state machine
described there.
```

Cross-reference when:

- A project-wing entry builds on a shared-wing entry. The project entry states only the
  local specifics; the shared rule is referenced, not duplicated.
- Two entries in different rooms of the same wing address related aspects and a reader of
  one benefits from knowing about the other.
- A `decision` records the reasoning behind an `invariant`, or vice versa.

Do **not** cross-reference:

- As a substitute for writing a real entry. A reference to an entry that does not exist
  is worse than no reference.
- To compensate for a compound entry that should have been split (invariant **I8**).

When your change makes an existing cross-reference invalid (e.g., you deprecate the
target), update or remove the referring entries.

---

## 8. How to resolve conflicts

Two entries conflict when they address the same topic and disagree on the rule.

**Step 1 — Classify the conflict:**

| Situation | Classification |
|-----------|---------------|
| Project entry and shared entry disagree, project entry has an explicit `local-override` marker | **Intentional override.** Project entry wins, per invariant **I2**. |
| Project entry and shared entry disagree, no override marker | **Drift or undocumented divergence.** Do not pick a winner. Surface the conflict. |
| Two entries in the same wing/room disagree | **Authoring failure.** One is stale, a duplicate, or a split that was never finished. |
| Two `invariant` entries disagree | **Governance failure.** Neither can be `active` until the conflict is resolved. |

**Step 2 — Act based on classification:**

- **Intentional override:** ensure the override entry states *why* (rationale is required
  under invariant **I3**). No resolution needed beyond verifying this marker.
- **Drift or undocumented divergence:** flag to a human. Do not modify either entry
  silently. Agents must not resolve this class of conflict on their own.
- **Authoring failure in one wing/room:** determine which entry reflects current reality.
  Mark the other `deprecated` with a `superseded_by` reference, per rule **§6**.
- **Conflicting invariants:** freeze both (`status: under_review`) until a human narrows
  scope, deprecates one, or replaces both with a single clearer invariant.

**Never:**

- Silently merge contradictory entries into one — the merge erases the conflict without
  resolving it.
- Pick the "more recent" entry by `created_at` alone. Recency is not authority.
- Treat a similarity score as a conflict resolver. Similarity flags candidates for
  review; it does not decide them.

See [docs/architecture.md §7 Conflict Handling](../docs/architecture.md#7-conflict-handling)
for the precedence rule that underpins Step 1.

---

## Quick Reference

| Question | Rule |
|----------|------|
| New scoping boundary? | §1 Create a wing |
| Existing room too broad? | §2 Create a room |
| Content is durable and non-obvious? | §3 Write an entry |
| Session-scoped, obvious, or speculative? | §4 Do not write memory |
| Related entry already exists? | §5 Enrich instead of creating |
| Entry no longer matches reality? | §6 Mark obsolete |
| Project entry depends on shared rule? | §7 Cross-reference |
| Entries disagree? | §8 Resolve conflicts |
