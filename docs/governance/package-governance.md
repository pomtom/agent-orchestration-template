# Governance: Dependencies & Packages

Operational policy for managing NuGet dependencies. The enforceable rules live in
[`docs/standards/package-governance-standard.md`](../standards/package-governance-standard.md);
this document covers process and ownership.

## Policy

- **Central Package Management is mandatory** — `Directory.Packages.props` +
  `Directory.Build.props` at the repo root; no inline versions in `.csproj`.
- **Pinned versions only** — no floating ranges in committed manifests.
- **Trusted feeds only** — restore via a controlled `nuget.config`.
- **No vulnerable or deprecated packages** may be merged.

## Cadence

| Activity | Frequency | How |
|---|---|---|
| Vulnerability scan | every PR | `dotnet list package --vulnerable` (CI) + `dependency-scanning` MCP |
| Deprecated check | weekly | `dotnet list package --deprecated` |
| Outdated review | monthly | `dotnet list package --outdated` + `package-governance` skill |
| Unused dependency sweep | quarterly | `package-governance` skill |

## Roles

- **Authors** keep dependencies centralized and justify new packages in the PR.
- **Reviewers** confirm the package-governance checklist item in the PR template.
- **Maintainers** own the upgrade backlog and security response.

## Upgrade rules

- Security patches: prioritize; merge promptly after CI passes.
- Minor/patch: batch routinely.
- **Major versions: treated as breaking** — require explicit review and a test pass.

## Adding a dependency — checklist

- [ ] Actively maintained, appropriately licensed, from a trusted feed
- [ ] No known vulnerabilities (scanned)
- [ ] Version added to `Directory.Packages.props` (not the `.csproj`)
- [ ] Justified in the PR description
