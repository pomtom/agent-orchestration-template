---
name: architecture-reviewer
description: Review a .NET solution's architecture and produce an actionable report — Clean Architecture compliance, SOLID, CQRS/MediatR, Repository pattern, dependency injection, layer separation, technical debt, security concerns, and performance bottlenecks. Use when the user asks for an architecture review, design assessment, tech-debt analysis, or SOLID/Clean-Architecture compliance check. For large solutions, delegate to the architecture-reviewer subagent.
---

# Architecture Reviewer

Assess architecture and report actionable findings. This skill **reviews**; it does not
rewrite business code.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   and emit the detection summary as the report header.
2. **Assess every dimension** in
   [architecture-standard](../../../docs/standards/architecture-standard.md): Clean
   Architecture/layering, SOLID, CQRS & MediatR, Repository/data access, DI lifetimes,
   cross-cutting concerns (exceptions/logging/config/async/correlation), technical debt,
   security, and performance.
3. **Gather evidence** — cite `file:line` for each finding.
4. **Produce the report** exactly as specified in the standard: detection summary,
   per-dimension scorecard, grouped findings (observation/impact/recommendation/effort/
   severity), and a prioritized top-10 remediation backlog.
5. **For large or multi-project solutions**, delegate the deep pass to the
   `architecture-reviewer` subagent (`.claude/agents/architecture-reviewer.md`) so the
   main context stays clean.

## Guardrails

- Recommendations only — **no business-code rewrites** unless the user explicitly asks a
  specific enforcer skill to apply a fix.
- Be specific and evidence-based; avoid generic advice not grounded in the code.
