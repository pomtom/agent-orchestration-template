---
name: naming-convention-enforcer
description: Enforce consistent .NET naming for projects, folders, files, namespaces, services, repositories, DTOs, requests, responses, commands, queries, and events per Microsoft conventions. Use when the user asks to review or fix naming, standardize naming conventions, or align names with .NET guidelines.
---

# Naming Convention Enforcer

Enforce consistent naming across the solution, aligned with Microsoft / .NET design
guidelines.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md)).
2. **Audit names** at every level against
   [naming-standard](../../../docs/standards/naming-standard.md): projects, folders,
   files, namespaces, interfaces, async suffixes, and the component patterns
   (Service/Repository/Dto/Request/Response/Command/Query/Handler/Event/Options/
   Middleware/Controller/Function/Orchestrator/Activity).
3. **Report** each violation as `file:line`, current name, suggested name, and the rule.
4. **Separate breaking renames.** List renames that cross public API boundaries
   (public types/members, DI registrations, serialized DTO property names, route
   names) under a **Breaking** heading — these need explicit approval.
5. **Apply** non-breaking renames directly (update all references); for breaking ones
   (public types/members, serialized contracts, routes) confirm first, then apply and
   update all references. Do not stop at a list of suggestions.

## Guardrails

- Renaming must update all references, DI registrations, and (carefully) serialized
  contracts. Never rename serialized members silently — it breaks wire compatibility.
- **Build gate:** after applying renames, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
