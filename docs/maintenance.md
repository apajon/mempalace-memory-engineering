# Maintenance: Memory Lifecycle and Upkeep

Memory that is written once and never reviewed becomes unreliable. This document describes the lifecycle of an entry and the work that keeps memory useful over time.

---

## Entry lifecycle

```text
draft -> active -> under_review -> active (updated)
                           \
                            -> deprecated
```

| Status | Meaning |
|--------|---------|
| `draft` | Being written or reviewed; not part of standard retrieval |
| `active` | Current and available for retrieval |
| `under_review` | Possibly stale; needs confirmation |
| `deprecated` | Superseded or invalidated; excluded from default retrieval |

Lifecycle transitions should be explicit.

---

## Maintenance schedule

### Weekly for active projects

- review new entries for clarity and type correctness
- check whether recent changes should have triggered supersession

### Monthly

- run a deduplication pass across active wings
- review old `verified_at` dates and mark uncertain entries `under_review`
- resolve `under_review` entries by updating or deprecating them

### On significant system changes

- audit the most affected wing
- mark potentially stale entries `under_review`
- update or deprecate them after confirming impact

### Before starting a new agent session

- confirm that relevant invariants are still current
- check whether unresolved `under_review` entries affect the session
- remove or deprecate entries that were only meant to be temporary

---

## Ownership

Each wing should have a human owner responsible for review quality.

That usually includes:

- approving new `invariant` and `decision` entries
- leading maintenance reviews
- resolving contradictions between entries

---

## Signs maintenance is needed

Maintenance is probably overdue when:

- the same task produces inconsistent agent behavior across sessions
- a human traces a bad decision to stale memory
- the same concept appears in multiple conflicting entries
- many entries have not been verified for a long time
- a wing keeps growing without cleanup or deprecation

These are signals, not rigid thresholds.