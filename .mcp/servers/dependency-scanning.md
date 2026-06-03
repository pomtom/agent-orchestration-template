# MCP Server: Dependency Scanning

**Purpose:** Software-composition analysis — check NuGet dependencies against a
vulnerability database (OSV) to surface vulnerable/at-risk packages. Backs the
`package-governance` skill and the `tech-debt-assessment` workflow.

| | |
|---|---|
| **Key (`.mcp.json`)** | `dependency-scanning` |
| **Command** | `npx -y @modelcontextprotocol/server-osv` |
| **Required env** | none (public OSV API) |
| **Scopes** | n/a |

## Setup

1. No credentials required for the public OSV database.
2. If unavailable in your registry, substitute another SCA MCP server and update
   `.mcp.json`.

## Notes

- Complements (does not replace) `dotnet list package --vulnerable` in
  `standards-ci.yml`; use both for defense in depth.
- Credential-free; safe to validate early.
