# Retrieval: Order, Scope, and Priority

Retrieval is not just search. It is the process by which an agent assembles the right memory before it answers a question, changes code, reviews a design, or writes a new entry.

If architecture defines how memory is stored, retrieval defines how that memory becomes usable.

---

## 1. Why retrieval matters

Good retrieval does two things at once:

- it brings in the rules that should shape the current task
- it keeps irrelevant memory out of the context window

Poor retrieval usually looks like one of these failures:

- the agent misses a constraint that should have applied
- the agent loads too much low-value context and buries the important rule
- the agent combines entries from the wrong scope and invents a contradiction
- the agent treats stale memory as current memory

Retrieval quality is therefore part of the memory model itself, not a convenience feature layered on top.

---

## 2. Retrieval model

Retrieval has two stages:

1. choose the right scope
2. apply entries in the right authority order

That order matters. A good retrieval system does not start by asking "what looks semantically similar?" It starts by asking:

- which wing is active
- which room is most likely relevant
- which entry types must be applied first

In practice, retrieval is healthiest when it is:

- scoped before it is widened
- ordered by authority, not similarity alone
- filtered to the smallest useful set
- resilient when the preferred source is unavailable

---

## 3. Retrieval order

Within retrieved memory, apply entries in this order:

1. **Invariants** — hard constraints
2. **Decisions** — current design direction
3. **Patterns** — reusable ways of solving the problem
4. **Notes** — supporting context

Entries with `status: deprecated` stay out of default retrieval unless explicitly requested.

The reasoning is straightforward:

- `invariant` answers what must not be violated
- `decision` answers which direction is currently authoritative
- `pattern` answers how similar problems are usually solved
- `note` answers what extra context may still help

Lower-priority entries must not silently override higher-priority ones.

### Example

If memory contains:

- one `invariant` saying callbacks must not block
- one `decision` saying this project uses a single-threaded executor by default
- one `pattern` showing how to offload long-running work
- one `note` describing a past debugging session

the note may add context, but it must never weaken the invariant or reinterpret the decision.

---

## 4. Scope

Retrieval is scoped to the active wing by default.

If an agent is working on a ROS 2 project task, it should start with the relevant project wing or the shared `ros2` wing rather than searching across unrelated memory.

Cross-wing retrieval is allowed, but it should be explicit.

In practice, retrieval usually starts from one of these scopes:

- a **project wing** such as `lifecore_ros2`
- a **shared wing** such as `ros2` or `react`
- a **specific room** such as `architecture` or `anti-patterns`

The active scope should be narrow by default. Widen it only when the task clearly crosses boundaries.

### Scope-first examples

If the task is "update a project-specific ROS 2 lifecycle node":

1. retrieve from the project wing first
2. retrieve from the shared `ros2` wing second
3. merge results
4. let project entries override shared entries only when the override is explicit

If the task is "explain general ROS 2 callback behavior", the project wing may be unnecessary.

If the task is "review whether a local project rule duplicates shared React guidance", both project and shared wings are necessary from the start.

---

## 5. Filtering

Within a scope, retrieval can be narrowed:

- **By room**
- **By type**
- **By status**

Default retrieval should focus on `active` entries and include `under_review` entries only when their uncertainty is made clear.

Useful defaults:

- retrieve by **room** when the task is tightly scoped
- retrieve by **type** when the task is evaluative, such as review or policy checking
- retrieve by **status** only when looking for stale or superseded content intentionally

### Conceptual query shapes

These are conceptual query shapes, not backend-specific syntax:

```text
project_wing=lifecore_ros2, room=architecture, type=invariant|decision, status=active
```

```text
shared_wing=ros2, room=anti-patterns, type=invariant, status=active
```

```text
project_wing=myapp, room=incident-log, status=under_review|active
```

```text
project_wing=myapp, room=architecture|contracts, type=decision|pattern, status=active
```

The goal is not to search everything. The goal is to retrieve the smallest set of entries that can still govern the task correctly.

---

## 6. Fallback order

Retrieval should degrade gracefully when the memory backend is unavailable, incomplete, or returns uncertain results.

Use this fallback order:

1. memory backend
2. in-repo architecture docs
3. README or other overview docs
4. workspace search

In shorthand:

```text
memory backend -> docs -> README -> workspace search
```

The agent should not stop working just because the preferred retrieval layer failed. Memory is an input, not a prerequisite.

### Fallback example

Suppose a task requires lifecycle rules, but the memory backend is down.

A sensible fallback is:

1. read [docs/architecture.md](docs/architecture.md)
2. read [README.md](../README.md) for the repo-level summary
3. search the workspace for lifecycle-specific guidance
4. continue the task while making the degraded retrieval path explicit

The fallback path should preserve momentum, not perfectly reproduce the backend.

---

## 7. Context-window strategy

Retrieved memory uses context space, so degradation should be deliberate.

When context gets tight:

- keep invariants first
- keep decisions in full when they directly shape the task
- summarize patterns if needed
- include notes only when they add direct value

This is not an optimization detail. It is a retrieval policy.

### Recommended strategy

1. Keep all applicable `invariant` entries.
2. Keep only the `decision` entries that shape the current task.
3. Summarize matching `pattern` entries into short operational guidance if needed.
4. Drop `note` entries unless they clarify uncertainty, explain a real exception, or prevent a likely mistake.

### Example under pressure

Suppose the agent is reviewing a change in a project wing with:

- 2 relevant invariants
- 3 active decisions
- 6 patterns
- 9 notes

Under context pressure, a good retrieval result may keep:

- both invariants in full
- the 2 decisions that directly constrain the touched component
- a short summary of the 2 most relevant patterns
- no notes at all

The point is not completeness. The point is preserving authoritative context.

### Practical compression rule

When reducing retrieval output, compress in this order:

1. drop irrelevant notes
2. summarize patterns
3. trim marginal decisions
4. widen only if the remaining context is still insufficient

Do not compress invariants away.

---

## 8. Task-oriented retrieval examples

### Code review

For a code review on a lifecycle component:

1. query `project_wing/architecture`
2. query `project_wing/anti-patterns`
3. query `shared_wing/architecture` or `shared_wing/conventions` if needed
4. apply invariants first
5. report any contradiction rather than resolving it silently

### Code change

For modifying a repository-layer component:

1. retrieve project `architecture` invariants and decisions
2. retrieve shared framework patterns only if the change depends on them
3. ignore incident logs unless the component has a known recurring failure
4. keep only the entries that constrain the touched code path

### New memory authoring

For writing a new memory entry:

1. retrieve the target room first
2. retrieve nearby rooms only if overlap seems likely
3. check the shared wing if the topic may be reusable
4. decide whether to enrich or create only after that read

### Explanation task

For explaining a general concept rather than editing code:

1. start with the shared wing
2. only bring in project memory if the user asks for project-specific behavior
3. prefer decisions and patterns over notes
4. include invariants whenever the explanation risks implying the wrong behavior

---

## 9. Retrieval failures

Retrieval usually fails in one of three ways:

- the agent uses an entry that does not exist
- the agent ignores an entry that does exist
- the agent applies an entry outside its scope

When that happens, the fix is usually to review:

- entry types
- scope assignment
- room placement
- status accuracy
- instruction-file behavior

### Common failure patterns

**Wrong-scope retrieval**

- symptom: the agent applies a shared convention where a project override exists
- likely cause: retrieval skipped the project wing or ignored the override marker
- fix: query the project wing first and treat undocumented contradictions as review issues

**Over-retrieval**

- symptom: the agent cites many loosely related entries but misses the key rule
- likely cause: the query was too broad or had no room filter
- fix: narrow by room and keep authority order visible

**Under-retrieval**

- symptom: the answer is clean but violates an invariant
- likely cause: invariant retrieval was not unconditional
- fix: make invariant retrieval mandatory for the active scope

**Stale retrieval**

- symptom: the agent relies on an entry that no longer matches the current system
- likely cause: `under_review` or `deprecated` handling is weak
- fix: strengthen status handling and maintenance review

**Similarity-led retrieval**

- symptom: a semantically similar note outranks a binding decision
- likely cause: retrieval was driven by similarity without respecting type order
- fix: resolve scope and type before using similarity as a ranking aid

---

## 10. Practical retrieval flow

For most tasks, this sequence is enough:

1. identify the active wing
2. identify the most likely room
3. retrieve `invariant` and `decision` entries first
4. add `pattern` entries only if they help with implementation or explanation
5. add `note` entries only if they clarify uncertainty
6. fall back to local docs and workspace search if needed
7. widen scope only when the task genuinely crosses boundaries

### Quick checklist

```text
[ ] Active wing identified
[ ] Likely room identified
[ ] Invariants retrieved first
[ ] Decisions retrieved next
[ ] Patterns added only when useful
[ ] Notes added only when justified
[ ] Deprecated entries excluded by default
[ ] Fallback path available if the backend is unavailable
```

---

## 11. Summary

Retrieval is healthy when it is:

- scoped before it is widened
- ordered by authority
- filtered to the smallest useful set
- resilient when the backend is unavailable
- explicit about context-window tradeoffs

If retrieval feels noisy, the fix is usually not to search harder. It is to improve scope, types, status handling, and fallback behavior.