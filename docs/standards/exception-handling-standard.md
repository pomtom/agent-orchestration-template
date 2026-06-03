# Standard: Exception Handling

> Canonical exception-handling rules. Consumed by the `exception-handling-enforcer` skill.

## Principles

1. **Centralize.** Use a single global handler per host; do not scatter try/catch for
   cross-cutting concerns.
   - **Web API / MVC / Blazor Server:** exception-handling middleware
     (`UseExceptionHandler` with a handler, or a custom
     `ExceptionHandlingMiddleware`) producing **ProblemDetails** (RFC 7807).
   - **Azure / Durable Functions (isolated):** a function **middleware**
     (`IFunctionsWorkerMiddleware`) that catches, logs, and shapes the response;
     for in-proc, a try/catch wrapper or filter.
   - **Worker / Console:** catch at the top of the unit of work / message loop; let
     truly fatal errors stop the host.
2. **Throw specific, meaningful exceptions.** Prefer domain exceptions
   (`OrderNotFoundException`, `InsufficientInventoryException`) over `Exception`,
   `ApplicationException`, or `InvalidOperationException` for domain conditions.
3. **Never swallow.** No empty `catch {}`. If you catch, you must handle, rethrow
   (`throw;` — never `throw ex;`), or wrap with an inner exception.
4. **Catch narrowly.** Avoid `catch (Exception)` except at the centralized boundary.
5. **Standardize API error responses.** All error responses use `ProblemDetails`
   with: `type`, `title`, `status`, `detail`, `instance`, and a `correlationId`
   extension. Map domain exceptions → HTTP status (validation→400, not-found→404,
   conflict→409, unauthorized→401/403, unexpected→500).
6. **Log every handled exception** at the boundary with structured context (see
   [logging-standard](./logging-standard.md)), including the correlation ID (see
   [correlation-id-standard](./correlation-id-standard.md)). Do not double-log the same
   exception at every layer.
7. **Do not leak internals.** Return generic messages + correlation ID to clients;
   keep stack traces and sensitive detail in logs only. Never include secrets/PII in
   messages.
8. **Validation vs. exceptions.** Prefer result/validation objects for expected
   business rule failures on hot paths; reserve exceptions for exceptional conditions.

## Recommended building blocks

- A small domain exception hierarchy (e.g., `DomainException` base →
  `NotFoundException`, `ValidationException`, `ConflictException`).
- An `ExceptionToProblemDetailsMapper` mapping exception types → status + title.
- Correlation ID surfaced in every error payload and log scope.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `catch (Exception) {}` | remove or handle/rethrow with logging |
| `throw ex;` | `throw;` (preserve stack) |
| `catch { throw new Exception("failed"); }` | wrap with inner: `throw new XException(msg, ex)` |
| returning 200 on error | map to correct status via ProblemDetails |
| try/catch in every method | move to centralized boundary |
| exception messages with PII/secrets | sanitize; log detail separately |

## Enforcement output

- Report file:line, the anti-pattern, the fix, and severity.
- Note where a centralized handler is missing for a detected host type and propose
  the appropriate middleware/wrapper.
