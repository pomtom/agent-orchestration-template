# MCP Server: Documentation Search

**Purpose:** Search authoritative documentation (Microsoft Learn / .NET / Azure docs) to
ground recommendations in current guidance — APIs, best practices, deprecations.

| | |
|---|---|
| **Key (`.mcp.json`)** | `documentation-search` |
| **Command** | `npx -y @microsoft/mcp-server-learn` |
| **Required env** | none (public docs) |
| **Scopes** | n/a |

## Setup

1. No credentials required.
2. If the Learn server isn't available in your registry, substitute an equivalent
   docs-search MCP server (e.g. context7) in `.mcp/configurations/` and update
   `.mcp.json`.

## Notes

- Credential-free; safe to validate early.
- Keeps standards advice aligned with current Microsoft guidance.
