# Maintenance: Memory Lifecycle and Upkeep

Memory that is written and never revisited becomes unreliable. This document describes the lifecycle of a memory entry and the maintenance practices that keep memory useful over time.

---

## Entry Lifecycle

```
draft → active → under_review → active (updated)
                              ↘ deprecated
```

| Status | Meaning |
|--------|---------|
| `draft` | Entry is being written; not yet in standard retrieval |
| `active` | Entry is current, verified, and available for retrieval |
| `under_review` | Entry may be stale or contested; flagged for human review |
| `deprecated` | Entry has been superseded or invalidated; excluded from standard retrieval |

Entries should move through this lifecycle explicitly. An entry should never become `deprecated` by being silently removed. The reason for deprecation and any replacement entry should be noted.

---

## Maintenance Schedule

Maintenance activities should be scheduled, not only reactive.

### Weekly (Active Projects)

- Review entries added in the past week for clarity and type correctness
- Identify any entries that should have triggered a supersession event but did not

### Monthly

- Run deduplication review across all active wings (see `docs/deduplication.md`)
- Check for entries with `verified_at` dates older than 60 days; flag as `under_review`
- Review `under_review` entries and either update or deprecate them

### On Significant System Changes

- Audit the wing most affected by the change
- Mark entries that may be affected as `under_review`
- Update or deprecate after confirming impact

### Before Starting a New Agent Session

- Confirm that invariants in the relevant wing are current
- Resolve any `under_review` entries that affect the session's scope
- Remove or deprecate entries that were known to be temporary

---

## Ownership

Each wing should have a designated owner — a person responsible for reviewing and approving changes to entries in that wing. Ownership is not exclusive authorship; it is accountability for quality.

Wing owners are responsible for:
- Approving new `invariant` and `decision` entries
- Conducting monthly maintenance reviews
- Resolving conflicts when entries contradict each other

---

## Signs That Maintenance Is Needed

- An agent produces inconsistent behavior across sessions for the same task
- A human engineer is surprised by an agent's behavior and traces it to an outdated entry
- The same concept appears in multiple entries with different content
- Several entries have `verified_at` dates more than 90 days old
- A wing has grown to more than 100 entries without any recent deprecations

These are signals, not thresholds. Use judgment. A wing with 200 carefully maintained entries is healthier than a wing with 30 entries that have never been reviewed.
