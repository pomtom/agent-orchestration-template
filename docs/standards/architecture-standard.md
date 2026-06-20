# Standard: Architecture Review

> Canonical architecture criteria. Consumed by the `architecture-reviewer` skill and
> the `architecture-reviewer` subagent. This is a **review/reporting** standard — it
> assesses, it does not rewrite business code.

## What to assess

### 1. Clean Architecture / layering
- Dependencies point inward: Domain has no dependency on Application/Infrastructure/UI.
- Application depends only on Domain (+ abstractions). Infrastructure implements
  abstractions defined inward. UI/host depends on Application, not on Infrastructure
  internals.
- No leakage of EF/HTTP/Azure SDK types into Domain.
- Flag layer-skipping and circular project references.

### 2. SOLID
- **SRP:** classes with one reason to change; flag god-classes/services.
- **OCP:** extension via abstractions, not modification of switch-walls.
- **LSP:** subtypes honor base contracts.
- **ISP:** focused interfaces; flag fat interfaces.
- **DIP:** depend on abstractions; concrete types injected, not `new`-ed across layers.

### 3. CQRS & MediatR (when present)
- Commands mutate, queries read; no queries with side effects.
- One handler per request; thin controllers/functions delegating to handlers.
- Pipeline behaviors for validation/logging/transaction where appropriate.
- If MediatR is **not** used, assess whether the equivalent separation exists; do not
  mandate MediatR — assess the pattern, not the library.
- *Apply/fix companion:* [cqrs-standard](./cqrs-standard.md) (`cqrs-pattern-enforcer`).

### 4. Repository / data access
- Repositories expose domain-meaningful operations, not leaky `IQueryable` everywhere.
- Unit of Work / transaction boundaries are clear.
- Flag business logic embedded in repositories or EF configurations.
- *Apply/fix companion:* [repository-pattern-standard](./repository-pattern-standard.md)
  (`repository-pattern-enforcer`).

### 5. Dependency Injection
- Correct lifetimes (singleton/scoped/transient); flag captive dependencies
  (singleton capturing scoped), `IServiceProvider` service-location abuse, and
  manual `new` of injectable services.
- Registrations grouped logically (per-module extension methods).
- *Apply/fix companion:* [dependency-injection-standard](./dependency-injection-standard.md)
  (`dependency-injection-enforcer`).

### 6. Cross-cutting
- Exception handling centralized ([exception-handling-standard](./exception-handling-standard.md)).
- Logging structured + correlated ([logging-standard](./logging-standard.md),
  [correlation-id-standard](./correlation-id-standard.md)).
- Configuration via Options ([configuration-standard](./configuration-standard.md)).
- Async correctness ([async-standard](./async-standard.md)).

### 7. Technical debt
- Duplication, dead code, large methods/classes, high coupling, missing tests,
  TODO/HACK markers, outdated/vulnerable packages
  ([package-governance-standard](./package-governance-standard.md)).

### 8. Security concerns
- AuthN/AuthZ coverage, secret management, input validation, injection surfaces,
  over-permissive CORS, sensitive data in logs, insecure deserialization.

### 9. Performance bottlenecks
- Sync-over-async, N+1 queries, unbounded in-memory loads, missing pagination,
  chatty I/O, missing caching where appropriate, large allocations on hot paths,
  Durable Functions fan-out without bounds.

## Output: actionable report

Produce a Markdown report with:
1. **Detection summary** (from [project-type-detection](./project-type-detection.md)).
2. **Scorecard** per dimension (Good / Needs work / At risk) with evidence (file:line).
3. **Findings** grouped by dimension, each with: observation, impact, recommendation,
   effort (S/M/L), and severity.
4. **Prioritized remediation backlog** (top 10), highest risk/▲value first.
5. Explicitly **no business-code rewrites** — recommendations only, unless the user
   asks a specific skill to apply a fix.
