# Example: Basic Two-Wing Memory Model

This example demonstrates a minimal memory structure for a small engineering project using two wings. It is intentionally simple and serves as a starting point for teams new to structured memory design.

---

## Project Context

A small team is building a backend service. They have two clear domains:

1. `architecture` — system design decisions, constraints, and patterns
2. `agent_behavior` — rules governing how AI agents should behave in this project

---

## Wing: `architecture`

### Room: `architecture/decisions`

**Entry: api-versioning-policy**
```
type: decision
status: active
created_at: 2025-01-10
verified_at: 2025-03-15

All public API endpoints must include a version prefix (e.g., /v1/, /v2/).
Breaking changes must be introduced under a new version.
The old version must remain supported for at least 90 days after the new version is released.

Rationale: External clients cannot be forced to upgrade on our timeline. Versioning gives them a migration window.
```

**Entry: database-migration-strategy**
```
type: decision
status: active
created_at: 2025-01-10
verified_at: 2025-03-15

All schema migrations must be backward-compatible for at least one release cycle.
Migrations that drop columns or tables require a separate deprecation release before removal.

Rationale: Zero-downtime deployments require that new code and old code can read the same schema simultaneously.
```

### Room: `architecture/invariants`

**Entry: no-direct-db-access-from-handlers**
```
type: invariant
status: active
created_at: 2025-01-10
verified_at: 2025-03-15

HTTP request handlers must not access the database directly.
All database access must go through the service layer.

This invariant must not be violated even for convenience or performance shortcuts.
```

---

## Wing: `agent_behavior`

### Room: `agent_behavior/instructions`

**Entry: code-review-scope**
```
type: invariant
status: active
created_at: 2025-01-12
verified_at: 2025-03-15

When reviewing code, the agent must:
1. Load the architecture/invariants room before commenting.
2. Flag any code that violates invariants as a blocking issue, not a suggestion.
3. Not approve a pull request that contains an invariant violation.
```

**Entry: migration-check**
```
type: pattern
status: active
created_at: 2025-01-12
verified_at: 2025-03-15

When a pull request includes a schema migration:
1. Check the migration for backward compatibility per the database-migration-strategy decision.
2. If the migration drops a column or table without a preceding deprecation release, flag it as a blocking issue.
```

---

## What This Model Demonstrates

- **Two wings, minimal rooms.** The structure is proportional to the project's complexity.
- **Invariants are in their own room.** This makes them easy to load as a batch during retrieval.
- **Agent instructions reference specific entries.** The `code-review-scope` invariant tells the agent exactly which memory to load and what to do with it.
- **Decisions include rationale.** The "why" is part of the entry, not separate documentation that may become disconnected.

---

## When to Add a Third Wing

Add a third wing when a new domain of concern emerges that would not fit cleanly into the existing wings without making them incoherent. In this example, adding `testing` as a wing makes sense once testing conventions and constraints warrant their own space. Adding `general` does not — "general" is a symptom of insufficient scoping.
