# Standard: Naming Conventions

> Canonical naming rules, aligned with Microsoft / .NET Framework Design Guidelines.
> Consumed by the `naming-convention-enforcer` skill.

## General casing

- **PascalCase:** types, methods, properties, events, namespaces, constants, enums.
- **camelCase:** local variables, method parameters.
- **`_camelCase`:** private instance fields. **`s_camelCase`** for private static
  fields (optional house style — match the existing codebase).
- **Interfaces** prefixed with `I` (`IOrderRepository`).
- **Generic type parameters** prefixed with `T` (`TKey`, `TResult`).
- **Async methods** suffixed with `Async`.
- No Hungarian notation; no abbreviations except well-known ones (Id, Http, Url, Xml).

## Solution-level naming

| Element | Convention | Example |
|---|---|---|
| **Project** | `Company.Product.Layer` | `Contoso.Orders.Api`, `Contoso.Orders.Application`, `Contoso.Orders.Domain`, `Contoso.Orders.Infrastructure` |
| **Folder** | PascalCase, matches namespace segment | `Features/Orders`, `Services` |
| **File** | matches the primary type name | `OrderService.cs` |
| **Namespace** | matches folder path under project root | `Contoso.Orders.Application.Features.Orders` |

## Component naming patterns

| Concept | Convention | Example |
|---|---|---|
| **Service** | `<Noun>Service` (+ `I<Noun>Service`) | `PricingService` / `IPricingService` |
| **Repository** | `<Entity>Repository` (+ interface) | `OrderRepository` / `IOrderRepository` |
| **DTO** | `<Name>Dto` | `OrderDto` |
| **Request** | `<Action><Entity>Request` | `CreateOrderRequest` |
| **Response** | `<Action><Entity>Response` | `CreateOrderResponse` |
| **Command** (CQRS) | `<Action><Entity>Command` | `CancelOrderCommand` |
| **Query** (CQRS) | `Get<Entity[Criteria]>Query` | `GetOrderByIdQuery` |
| **Command/Query handler** | `<CommandOrQuery>Handler` | `CancelOrderCommandHandler` |
| **Event** | past-tense `<Entity><Verb>edEvent` | `OrderCancelledEvent` |
| **Options** | `<Area>Options` | `StorageOptions` |
| **Middleware** | `<Concern>Middleware` | `ExceptionHandlingMiddleware` |
| **Controller** | `<Resource>Controller` | `OrdersController` |
| **Azure Function** | `<Verb><Noun>Function` or trigger-descriptive | `ProcessOrderFunction` |
| **Durable orchestrator** | `<Process>Orchestrator` | `OrderFulfillmentOrchestrator` |
| **Durable activity** | `<Verb><Noun>Activity` | `ReserveInventoryActivity` |

## Other rules

- Booleans read as assertions: `IsEnabled`, `HasItems`, `CanRetry`.
- Collections are plural (`orders`), singletons singular (`order`).
- Enums singular for values, plural only for `[Flags]`.
- Avoid stutter: `Order.OrderId` → prefer `Order.Id`.
- Test classes/methods follow [testing-standard](./testing-standard.md).

## Enforcement output

- Report each violation as file:line, current name, suggested name, and rule.
- Renames that cross public API boundaries (public types/members, DI registrations,
  serialized DTO names) must be flagged as breaking and listed separately.
