---
name: repository-pattern-enforcer
description: Enforce correct Repository & Unit of Work usage in .NET data access — domain-meaningful per-aggregate repositories, no leaky IQueryable, explicit unit-of-work commit boundaries, abstractions owned by the inner layer, no DbContext injected into handlers, and async/cancellation-aware queries. Avoids the blanket generic-repository anti-pattern over EF Core. Use when the user asks to review or fix repositories, add a repository/unit-of-work layer, stop leaking IQueryable/DbContext, or clean up data access.
---

# Repository & Unit of Work Enforcer

Drive data access toward intention-revealing, per-aggregate repositories with an explicit
unit-of-work commit boundary — and away from leaky or blanket-generic abstractions.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   and the data-access stack (EF Core, Dapper, Azure SDK) so the fixes match.
2. **Audit** against
   [repository-pattern-standard](../../../docs/standards/repository-pattern-standard.md):
   `IQueryable<T>` returned from repositories, blanket `IRepository<T>` over every entity,
   `SaveChangesAsync` called inside each repository method instead of one unit-of-work
   commit, `DbContext` injected directly into handlers/services, repository interfaces
   defined in Infrastructure instead of the inner layer, business rules embedded in
   queries, unbounded in-memory loads, and missing `CancellationToken`/async EF APIs
   ([async-standard](../../../docs/standards/async-standard.md)).
3. **Report** `file:line`, the anti-pattern, the fix, and severity.
4. **Apply** mechanical, behavior-preserving refactors directly: replace `IQueryable`
   returns with materialized/paged results, move the commit to a single unit-of-work call
   per operation, depend on the repository/UoW abstraction instead of `DbContext`, and
   relocate interfaces inward. Pause for approval where a public signature or transaction
   boundary changes observable behavior.

## Guardrails

- **Do not impose a blanket generic `IRepository<T>`** over EF Core — `DbContext` is
  already a unit of work and `DbSet<T>` a generic repository. Prefer explicit per-aggregate
  repositories; a generic base is acceptable only for simple read models/CRUD.
- Repositories do persistence only — no domain rules, no DTO mapping.
- Commit once per operation; with CQRS the commit lives in a `TransactionBehavior`
  ([cqrs-standard](../../../docs/standards/cqrs-standard.md)). Lifetimes are **scoped**
  ([dependency-injection-standard](../../../docs/standards/dependency-injection-standard.md)).
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
