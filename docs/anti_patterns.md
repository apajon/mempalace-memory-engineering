# Anti-Patterns in Memory Engineering

This document describes common failure modes in AI memory design. Recognizing these patterns early prevents the gradual decay of memory quality that makes agents unreliable.

---

## Flat Memory

**Description:** All entries are stored at the same level without wing or room structure. Memory is a list.

**Why it fails:** Retrieval has no way to scope or prioritize. As volume grows, every retrieval returns too much noise or too little signal. There is no way to mark something as a constraint versus a casual observation.

**Fix:** Define wings based on domains, rooms based on sub-topics, and assign types to every entry before storing it.

---

## Shadow Memory

**Description:** An agent is given instructions in a system prompt or conversation that contradict or override entries in persistent memory, without updating persistent memory.

**Why it fails:** The next session starts fresh from persistent memory. The override is gone. The agent behaves as if the override never happened. Over time, persistent memory and actual system behavior diverge.

**Fix:** Any instruction that should survive a session must be written to persistent memory. System prompts should reference memory, not replace it.

---

## Orphaned Entries

**Description:** Entries exist in memory for concepts or components that no longer exist in the system.

**Why it fails:** Agents draw on orphaned entries as if they are current. They make decisions based on constraints for systems that were removed.

**Fix:** When a component or subsystem is removed or significantly refactored, audit the relevant wing and deprecate all entries that no longer apply.

---

## Vague Invariants

**Description:** Invariants are written as general preferences rather than specific constraints.

**Example of a vague invariant:** "Prefer clean code."

**Why it fails:** The agent cannot operationalize vague invariants. They provide no actionable guidance and crowd out useful entries.

**Fix:** Invariants should be specific, verifiable, and actionable. A good invariant specifies exactly what must or must not happen, in what context, and why (if not obvious).

**Example of a specific invariant:** "All ROS 2 nodes must call `rclcpp::shutdown()` before the process exits. Do not suppress this call even in error paths."

---

## Untimed Notes

**Description:** Notes are written without creation or verification timestamps.

**Why it fails:** Without dates, there is no way to detect drift. An untimed note is equally likely to be from last week or two years ago.

**Fix:** Every entry should carry `created_at` and `verified_at` metadata. Even if the backend does not enforce this, include it in the entry body.

---

## Memory Hoarding

**Description:** Entries are never deprecated or removed. Memory only grows.

**Why it fails:** Context pressure increases. Retrieval returns more and more entries, many of which are outdated. The ratio of useful to useless entries declines.

**Fix:** Deprecate entries as part of maintenance. Deprecated entries can be retained for traceability but should not appear in standard retrieval.

---

## Implicit Cross-Wing Dependencies

**Description:** An entry in one wing implicitly depends on a constraint defined in another wing, but no reference exists between them.

**Why it fails:** When the constraint in the source wing changes, the dependent entry is not updated. The dependency is invisible.

**Fix:** Make cross-wing dependencies explicit. If an entry in `ros2/nodes` depends on a constraint in `architecture/decisions`, add a reference. When the referenced entry changes, dependent entries can be reviewed.

---

## Agent-Authored Invariants Without Review

**Description:** An agent is allowed to create or modify `invariant`-type entries without human approval.

**Why it fails:** Invariants constrain the agent's own behavior. Allowing agents to self-modify constraints creates a feedback loop where the agent can reduce its own constraints over time.

**Fix:** Invariant and decision entries should require human review before becoming `active`. Agents may draft entries, but a human should approve them before they enter the retrieval system with full weight.
