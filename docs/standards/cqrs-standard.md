# Standard: CQRS & Mediator

> Canonical command/query-separation rules. Consumed by the `cqrs-pattern-enforcer`
> skill. This is an **enforce/apply** standard — it introduces and corrects the pattern.
> Its read-only companion is [architecture-standard §3](./architecture-standard.md), which
> *assesses* CQRS; this standard *applies* it. Keep the two consistent.

## Principles

1. **Separate writes from reads.** A **Command** changes state and returns void or a
   minimal result (id/status); a **Query** returns data and **must not mutate state**.
   No query has side effects; no command doubles as a read model.
2. **One handler per request.** Each command/query is a distinct request type with exactly
   one handler (`CreateOrderCommand` → `CreateOrderHandler`). Handlers are the unit of
   application logic; do not branch one handler across multiple operations.
3. **Thin entry points.** Controllers, minimal-API endpoints, Function triggers, and
   Blazor components **delegate** to a handler and translate the result to a response.
   They contain no business logic, no data access, and no orchestration beyond mapping.
4. **Library-agnostic.** MediatR is a common dispatcher but **not required** (and recent
   versions are commercially licensed). Enforce the *pattern* — request/handler
   separation, one handler per request, pipeline cross-cutting — whether the repo uses
   MediatR, a hand-rolled `ISender`/dispatcher, or direct handler injection. Do not add
   or mandate a specific library; match what the solution already uses.
5. **Cross-cutting via pipeline behaviors.** Validation, logging, correlation-id scoping,
   transactions, caching, and retries belong in **pipeline behaviors** (or decorators when
   no mediator is present) — not copied into every handler. Order matters:
   logging → validation → transaction → handler.
6. **Requests and results are immutable DTOs.** Commands/queries are `record`/`sealed`
   types carrying only input data; results are dedicated response types. Do not pass
   domain entities or EF-tracked types across the handler boundary into the UI.
7. **Naming follows the convention.** `<Verb><Noun>Command` / `<Get|List><Noun>Query`,
   `<Request>Handler`, `<Noun>Response`/`<Noun>Dto` — see
   [naming-standard](./naming-standard.md).
8. **Expected failures are results, not exceptions.** Validation and business-rule
   failures on the hot path return a result/`ValidationException`-mapped response; reserve
   thrown exceptions for genuinely exceptional conditions
   ([exception-handling-standard](./exception-handling-standard.md)).

## Recommended building blocks (introduce only what's missing)

- `ICommand`/`ICommand<TResult>` and `IQuery<TResult>` marker interfaces (or the mediator's
  `IRequest<T>`), with matching `ICommandHandler<,>`/`IQueryHandler<,>`.
- A `ValidationBehavior` running the request's validators before the handler.
- A `LoggingBehavior` opening a correlation-id scope and timing the handler.
- A `TransactionBehavior` (commands only) wrapping the unit-of-work commit
  ([repository-pattern-standard](./repository-pattern-standard.md)).

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| Query method that writes/saves | move the write into a command; keep the query read-only |
| Business logic in controller/Function/component | delegate to a handler; entry point only maps |
| One "god" handler with a `switch` over operations | split into one handler per request |
| Domain entity / `DbContext` type returned to the UI | return a `Response`/`Dto` |
| Validation duplicated in every handler | a single `ValidationBehavior`/decorator |
| `try/catch` + transaction copied into each handler | a `TransactionBehavior` |
| Handler calling another handler directly | extract shared logic to a domain/app service |

## Project-type notes (after [detection](./project-type-detection.md))

- **Web API / MVC / Blazor:** action/endpoint/component sends the request and maps the
  result to `IActionResult`/markup; no data access in the entry point.
- **Azure / Durable Functions:** the trigger method sends the request; in **Durable
  orchestrators** keep dispatch deterministic — orchestration logic stays in the
  orchestrator, business logic in activity-invoked handlers.
- **Worker / Console:** the message-loop / job sends the command per unit of work.

## Enforcement output

- Report `file:line`, the anti-pattern, the fix, and severity.
- When the separation is absent, propose the minimal building blocks above and show the
  thin entry point + handler split. Apply mechanical, behavior-preserving extractions
  directly; pause for approval on changes to public endpoints or serialized contracts.
- **Build gate:** rebuild after changes per
  [build-verification-standard](./build-verification-standard.md).
