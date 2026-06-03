# MCP Server: Code Search

**Purpose:** Fast, local code/file search and reading across the repository, scoped to
the workspace root. Used by enforcer skills and reviewers to locate patterns
(`.Result`, raw `IConfiguration`, missing correlation handling, etc.).

| | |
|---|---|
| **Key (`.mcp.json`)** | `code-search` |
| **Command** | `npx -y @modelcontextprotocol/server-filesystem ${WORKSPACE_ROOT}` |
| **Required env** | `WORKSPACE_ROOT` (absolute path to the repo) |
| **Scopes** | Read access to the workspace path |

## Setup

1. Set `WORKSPACE_ROOT` to the repo root (e.g. the host repository where the framework
   is dropped in).
2. No credentials required — runs fully locally.

## Notes

- Credential-free: ideal first server to validate `.mcp.json` wiring.
- Scope the path narrowly; do not point it at your whole drive.
