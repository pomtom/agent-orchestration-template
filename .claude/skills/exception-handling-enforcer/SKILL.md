---
name: exception-handling-enforcer
description: Enforce centralized, consistent exception handling in .NET — exception middleware, meaningful domain exception types, no empty catches or throw ex, standardized ProblemDetails API error responses, correlation IDs, and proper logging. Use when the user asks to review or fix error handling, add a global exception handler, or standardize API error responses.
---

# Exception Handling Enforcer

Drive exception handling toward a centralized, meaningful, observable model.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to choose the right centralized mechanism (ASP.NET middleware, Functions worker
   middleware, worker/console boundary).
2. **Audit** against
   [exception-handling-standard](../../../docs/standards/exception-handling-standard.md):
   missing centralized handler, `catch (Exception){}`, `throw ex;`, generic catches,
   swallowed exceptions, non-standard error responses, exceptions without logging,
   missing correlation ID, and leaked internals/PII.
3. **Report** `file:line`, anti-pattern, fix, severity.
4. **Propose building blocks** when missing: domain exception hierarchy, exception→
   ProblemDetails mapper, and the host-appropriate centralized handler that logs with
   the correlation ID (see
   [correlation-id-standard](../../../docs/standards/correlation-id-standard.md) and
   [logging-standard](../../../docs/standards/logging-standard.md)).
5. **Apply** the centralized handler + mechanical fixes directly (`throw ex;`→`throw;`,
   remove empty catches, introduce domain exceptions). Do not stop at a list of
   suggestions; confirm first only for behavior-changing edits.

## Guardrails

- Error responses must not leak stack traces/secrets; client gets generic message +
  correlation ID, detail goes to logs.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
