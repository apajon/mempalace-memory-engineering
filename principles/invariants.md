# Invariants: What They Are and How to Define Them

An invariant is a constraint that must hold true at all times within a defined scope. In memory engineering, invariants are the highest-weight entry type. They represent rules that the agent must never violate, regardless of context, convenience, or user instruction — unless a human explicitly authorizes an exception.

---

## What Makes Something an Invariant

An invariant is not a preference or a best practice. It is a constraint with consequences if violated. The test for whether something should be an invariant:

1. **If this is violated, is the system in a bad state?** A violated invariant should produce a concrete bad outcome: incorrect behavior, safety risk, broken contracts with external systems, or unacceptable technical debt.
2. **Is it specific enough to be checked?** Vague statements like "prefer quality" are not invariants. A valid invariant specifies what must or must not happen, in what context, and leaves no room for ambiguous interpretation.
3. **Should it hold across all sessions and all agents working in this wing?** If an invariant applies only in one session or is easily overridden by context, it is probably a `decision` or `pattern`, not an invariant.

---

## Invariant Scope

Invariants are scoped to a wing. An invariant in the `ros2` wing applies to all agents operating in the `ros2` wing. It does not automatically apply to other wings.

If a constraint must apply globally across all wings, it belongs in a dedicated `global` wing or `agent_behavior` wing, depending on whether it governs system behavior or agent behavior.

---

## How to Write an Invariant

A well-formed invariant includes:

- **What must or must not happen** — the constraint itself, stated directly
- **In what context** — the scope within which the constraint applies
- **Why** (when non-obvious) — the consequence of violation, stated plainly

### Template

```
[What must/must not happen] in [context].
[Optional: Why — consequence of violation.]
```

### Examples

**Good:**
```
All database writes must go through the repository layer. Direct ORM access from service
methods is not permitted.

Reason: Direct ORM access bypasses audit logging and transaction management enforced
by the repository layer.
```

**Too vague (not a valid invariant):**
```
Keep code clean and maintainable.
```

**Falsely specific (not a valid invariant — this is a pattern):**
```
When possible, use list comprehensions instead of for-loops in Python.
```

---

## Invariant Review and Approval

Because invariants constrain agent behavior at the highest weight, they must be reviewed and approved by a human before becoming `active`. An agent may draft an invariant, but a human must explicitly approve it.

This prevents:
- Agents adding invariants that reduce their own constraints
- Invariants being added without understanding their full scope
- Conflicting invariants that cancel each other out

---

## Conflicting Invariants

If two invariants appear to conflict, the conflict must be resolved before either is marked `active`. Conflicting invariants produce undefined agent behavior. The resolution must be documented: one invariant may be narrowed in scope, one may be deprecated, or a new invariant may replace both with a clearer statement.

---

## Invariants Are Not Instructions

Agent instructions tell an agent what to do. Invariants tell the agent what must always be true. The distinction matters:

- Instructions may vary by task or session.
- Invariants do not vary. They apply unconditionally.

An agent that treats invariants as optional instructions is not using memory correctly.
