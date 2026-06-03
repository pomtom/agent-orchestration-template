# Workflow: Tech-Debt Assessment

Produce a prioritized technical-debt and risk picture for a .NET solution, combining
architecture, dependency, and security signals.

## When to use

Planning a hardening/modernization effort, a quarterly health check, or scoping
remediation work.

## Steps

1. **Detect project types** (`docs/standards/project-type-detection.md`).

2. **Deep architecture review.**
   Run the **`architecture-reviewer`** subagent → scorecard + findings across layering,
   SOLID, CQRS, DI, cross-cutting concerns, debt, security, performance
   (`docs/standards/architecture-standard.md`).

3. **Dependency governance.**
   Run the **`package-governance`** skill → vulnerable / deprecated / outdated / unused
   / not-centralized findings (`docs/standards/package-governance-standard.md`). Use the
   **Dependency Scanning** and **Security Scanning** MCP servers (`.mcp.json`) when
   available for deeper SCA results.

4. **Security pass.**
   Use the **Security Scanning** MCP server and the security dimension of the
   architecture review to surface auth gaps, secret handling, injection surfaces, and
   sensitive-data-in-logs issues. Cross-check `docs/governance/security-policy.md`.

5. **Consolidate & prioritize.**
   Merge findings into one backlog scored by risk × value × effort (S/M/L). Group into
   **Now / Next / Later**.

## Output

- A single tech-debt report: architecture scorecard, dependency/security findings, and a
  prioritized Now/Next/Later remediation backlog. Recommendations only — fixes are
  applied via the individual enforcer skills on approval.
