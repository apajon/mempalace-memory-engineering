# Anti-Patterns in Memory Engineering

This document names common ways memory systems become unreliable over time.

---

## Flat memory

**Description:** everything is stored at the same level, with no wing or room structure.

**Why it fails:** useful signal gets buried in undifferentiated context.

**Fix:** separate memory by wing, organize it by room, and assign a clear type to each durable entry.

---

## Shadow memory

**Description:** prompts or session instructions contradict persistent memory, but persistent memory is never updated.

**Why it fails:** the next session starts from stale persistent memory again.

**Fix:** anything that must survive a session should be written to persistent memory.

---

## Orphaned entries

**Description:** entries remain for systems or assumptions that no longer exist.

**Why it fails:** the agent keeps applying rules for a reality that is already gone.

**Fix:** audit the relevant wing when a component is removed or heavily refactored.

---

## Vague invariants

**Description:** invariants are written as preferences instead of operational constraints.

**Bad example:** "Prefer clean code."

**Why it fails:** the agent cannot act on vague advice.

**Fix:** write invariants as clear, testable statements.

**Better example:** "All ROS 2 nodes must call `rclcpp::shutdown()` before process exit, including error paths."

---

## Untimed notes

**Description:** entries are written without timestamps.

**Why it fails:** reviewers cannot tell whether an entry is fresh or ancient.

**Fix:** durable entries should include `created_at`, and entries that may drift should usually include `verified_at`.

---

## Memory hoarding

**Description:** memory only grows. Nothing gets deprecated or reviewed.

**Why it fails:** retrieval gets more expensive, more stale, and less trustworthy.

**Fix:** deprecate obsolete entries and make maintenance part of the normal workflow.

---

## Implicit cross-wing dependencies

**Description:** one entry depends on another wing's rule, but that dependency is never referenced.

**Why it fails:** the dependent entry will not be reviewed when the source rule changes.

**Fix:** cross-reference shared rules instead of copying them or depending on them silently.

---

## Agent-authored invariants without review

**Description:** an agent is allowed to activate its own `invariant` entries without human approval.

**Why it fails:** the agent can gradually reshape its own constraints.

**Fix:** agents may draft invariants and decisions, but a human should approve them before they become `active`.