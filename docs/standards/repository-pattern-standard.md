# Standard: Repository & Unit of Work

> Canonical data-access-abstraction rules. Consumed by the `repository-pattern-enforcer`
> skill. **Enforce/apply** companion to [architecture-standard §4](./architecture-standard.md)
> (which *assesses* data access). Keep the two consistent.

## Principles

1. **Repositories expose domain-meaningful operations, not a leaky data layer.** Methods
   read like the domain: `GetByIdAsync`, `FindActiveByCustomerAsync`,
   `AddAsync`, `Remove`. Do **not** return `IQueryable<T>` from a repository — that leaks
   the persistence model and lets query logic sprawl across the app.
2. **Do not impose a blanket generic `IRepository<T>` over EF Core.** `DbContext` is
   *already* a Unit of Work, and `DbSet<T>` is *already* a generic repository. A generic
   `IRepository<T>` with `GetAll`/`Find(Expression)` on top of EF is a recognized
   anti-pattern: it hides EF's capabilities, encourages `IQueryable` leakage, and adds a
   layer that earns nothing. **Prefer explicit, intention-revealing repositories per
   aggregate.** A generic base is acceptable **only** for simple read models / thin CRUD
   where no domain behavior exists — and even then it is a base class, not the public
   contract handlers depend on.
3. **One repository per aggregate root.** Repositories deal in aggregate roots and return
   fully-loaded domain objects. Don't create a repository per table or per entity that
   isn't an aggregate root.
4. **Explicit Unit of Work / transaction boundary.** Writes are committed through a single
   `IUnitOfWork.SaveChangesAsync(ct)` (or `DbContext.SaveChangesAsync`) at the **end of the
   operation** — not after every mutation inside a repository. Repositories *register*
   changes; the unit of work *commits* them. With CQRS, the commit lives in a
   `TransactionBehavior` ([cqrs-standard](./cqrs-standard.md)), not in each handler.
5. **Repositories are interfaces owned by the inner layer.** Define `IOrderRepository` in
   Domain/Application; implement it in Infrastructure. Handlers/services depend on the
   abstraction, never on `DbContext` directly ([architecture-standard](./architecture-standard.md)).
6. **No business logic in repositories or EF configurations.** Repositories do persistence
   only — no domain rules, no orchestration, no mapping to DTOs (that belongs in the
   handler/mapper). Flag domain decisions embedded in queries.
7. **Async + cancellation + no surprise materialization.** All I/O methods are `async`,
   accept and flow a `CancellationToken` ([async-standard](./async-standard.md)), and use
   async EF APIs (`ToListAsync`, `FirstOrDefaultAsync`, `SaveChangesAsync`). Avoid loading
   unbounded sets; expose paging parameters instead of returning everything.
8. **Read vs. write paths may differ.** Command-side repositories return tracked aggregate
   roots; query-side read models may bypass repositories entirely with projection
   (`Select` to a DTO) for performance. Don't force reads through write-side repositories.

## Recommended building blocks (introduce only what's missing)

- `IOrderRepository` (per aggregate) in the inner layer + an EF implementation in
  Infrastructure.
- `IUnitOfWork` with `Task<int> SaveChangesAsync(CancellationToken)` — often the
  `DbContext` itself behind an interface — committed once per operation.
- An optional generic `RepositoryBase<TEntity>` **for simple CRUD/read models only**,
  never returning `IQueryable` to callers.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `IQueryable<T> GetAll()` on a repository | return materialized domain objects or a paged result |
| Blanket `IRepository<T>` over every EF entity | explicit per-aggregate repositories |
| `SaveChangesAsync` called inside each repository method | commit once via the unit of work at operation end |
| `DbContext` injected directly into handlers/services | depend on the repository/UoW abstraction |
| Domain rules inside a repository query | move rules to the domain/handler |
| Repository returning EF-tracked entities to the UI | map to a `Dto`/`Response` |
| Loading whole tables into memory then filtering in C# | push the predicate/paging to the query |
| Repository interface defined in Infrastructure | define it inward (Domain/Application) |

## Project-type notes (after [detection](./project-type-detection.md))

- **Web API / MVC / Blazor / Worker / Console:** repositories + UoW injected as **scoped**
  services ([dependency-injection-standard](./dependency-injection-standard.md)); commit
  per request / per unit of work.
- **Azure / Durable Functions:** scope the `DbContext`/UoW per function invocation; in
  **Durable activities** keep each activity's data access self-contained and committed
  within the activity — never share a tracked context across orchestrator awaits.

## Enforcement output

- Report `file:line`, the anti-pattern, the fix, and severity.
- When abstractions are missing, propose the per-aggregate repository interface + EF
  implementation + unit-of-work commit point. Apply mechanical, behavior-preserving
  refactors directly; pause for approval where a public signature or transaction boundary
  changes observable behavior.
- **Build gate:** rebuild after changes per
  [build-verification-standard](./build-verification-standard.md).
