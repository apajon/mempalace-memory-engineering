# Deduplication and Drift Control

Over time, memory accumulates redundancy and drift. Deduplication is the process of identifying and resolving redundant entries. Drift control is the process of detecting and correcting entries that no longer reflect the current system state.

Both are ongoing maintenance concerns, not one-time tasks.

---

## Deduplication

### What Deduplication Addresses

Duplication occurs when:

- The same concept is recorded in multiple entries, often with slightly different wording
- A new entry is added without checking whether an existing entry already covers the topic
- A decision is re-recorded after being forgotten or overlooked

Duplicate entries create ambiguity. When two entries address the same concept with different content, an agent cannot reliably determine which is authoritative. This leads to inconsistent behavior.

### How to Deduplicate

1. **Identify candidates** — Entries within the same room that cover similar topics are the most common sources of duplication. Cross-room duplication also occurs when the same pattern is recorded independently in multiple contexts.
2. **Determine authority** — For each set of duplicates, determine which entry is most accurate and complete. Check `verified_at` dates and content quality.
3. **Merge or remove** — If one entry supersedes the others, mark the others `deprecated` with a reference to the surviving entry. If entries are complementary, consolidate them into a single entry.
4. **Do not silently delete** — Deprecated entries should be retained for traceability. Mark them clearly and do not include them in standard retrieval.

### Deduplication Schedule

Deduplication should be performed:

- After any significant batch of new entries is added
- Before a new agent session begins work in a wing
- On a regular schedule (e.g., monthly for active projects)

---

## Drift Control

### What Drift Is

Drift occurs when an entry's content no longer reflects the current state of the system it describes. Common causes:

- The system changed but memory was not updated
- A decision was revisited and reversed without updating the relevant entry
- An invariant was relaxed or tightened but the entry still reflects the old constraint

Drifted entries are more dangerous than missing entries. A missing entry causes the agent to ask or reason from first principles. A drifted entry causes the agent to act on incorrect information with confidence.

### Detecting Drift

Every entry should carry a `verified_at` date. An entry is considered **stale** when its `verified_at` date exceeds a defined threshold (e.g., 90 days for active projects).

Stale entries should be flagged for review. Review means a human or an authorized agent confirms that the entry still accurately reflects the system. If it does, update `verified_at`. If it does not, update the content or mark it `deprecated`.

### Supersession

When a decision or invariant changes:

1. Mark the old entry `deprecated` with a note referencing the replacement.
2. Create the new entry with a note referencing what it supersedes.
3. Do not delete the old entry. Traceability of why a decision changed is itself useful memory.

---

## Summary

| Process | Trigger | Outcome |
|---------|---------|---------|
| Deduplication | New batch of entries, scheduled review | Redundant entries merged or deprecated |
| Staleness check | Regular schedule, pre-session | Stale entries flagged for review |
| Supersession | Decision or invariant changes | Old entry deprecated, new entry created with back-reference |
