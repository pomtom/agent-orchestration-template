---
name: cqrs-pattern-enforcer
description: Enforce CQRS / command-query separation in .NET — commands mutate and queries read with no side effects, one handler per request, thin controllers/Functions/components that delegate to handlers, immutable request/response DTOs, and cross-cutting concerns in pipeline behaviors. Library-agnostic (works with or without MediatR). Use when the user asks to review or fix CQRS, separate reads from writes, thin out fat controllers, introduce handlers/mediator, or add validation/logging/transaction pipeline behaviors.
---

# CQRS / Mediator Pattern Enforcer

Drive application logic toward clean command/query separation with thin entry points and
one handler per request — without mandating a specific library.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   — note the entry-point shapes (controllers, minimal-API endpoints, Function triggers,
   Blazor components) that should delegate to handlers.
2. **Detect the existing dispatch style** — MediatR, a hand-rolled `ISender`/dispatcher,
   or direct handler injection. **Match it**; do not introduce or swap libraries (MediatR
   is now commercially licensed — never add it where it isn't already used).
3. **Audit** against
   [cqrs-standard](../../../docs/standards/cqrs-standard.md): queries with side effects,
   business logic in controllers/Functions/components, god-handlers branching over
   operations, domain/EF types leaked to the UI, validation/transaction/logging copied
   into handlers instead of pipeline behaviors, and naming drift
   ([naming-standard](../../../docs/standards/naming-standard.md)).
4. **Report** `file:line`, the anti-pattern, the fix, and severity.
5. **Apply** mechanical, behavior-preserving changes directly: extract logic out of entry
   points into handlers, split god-handlers, introduce missing `ValidationBehavior`/
   `LoggingBehavior`/`TransactionBehavior`, and replace leaked entities with response DTOs.
   Pause for approval on changes to **public endpoints or serialized contracts**.

## Guardrails

- Library-agnostic: enforce the pattern, not MediatR. Match the repo's dispatch style.
- Keep commands and queries strictly separated — never let a query mutate state.
- Transaction/commit boundaries belong in a behavior, coordinated with the unit of work
  ([repository-pattern-standard](../../../docs/standards/repository-pattern-standard.md)).
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
