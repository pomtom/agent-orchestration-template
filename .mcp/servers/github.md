# MCP Server: GitHub

**Purpose:** Read/write GitHub from agents — issues, PRs, code search, repo metadata,
Actions. Powers PR creation, issue triage, and cross-repo code search in workflows.

| | |
|---|---|
| **Key (`.mcp.json`)** | `github` |
| **Command** | `npx -y @modelcontextprotocol/server-github` |
| **Required env** | `GITHUB_PERSONAL_ACCESS_TOKEN` |
| **Scopes** | `repo` (private) or `public_repo`; add `workflow` for Actions, `read:org` for org data |

## Setup

1. Create a fine-grained or classic PAT at https://github.com/settings/tokens with the
   least scopes needed.
2. Export `GITHUB_PERSONAL_ACCESS_TOKEN` (see `.mcp/configurations/.env.example`).
3. Verify: `claude mcp list` then exercise a read-only call.

## Notes

- Credential-light to validate: a read-only token is enough for a smoke test.
- Never commit the token; use environment variables or your secret store.
