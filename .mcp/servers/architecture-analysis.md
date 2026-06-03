# MCP Server: Architecture Analysis

**Purpose:** A filesystem-backed surface scoped to source and architecture docs so the
`architecture-reviewer` skill/subagent can read code structure, project references, and
diagrams without broad filesystem access.

| | |
|---|---|
| **Key (`.mcp.json`)** | `architecture-analysis` |
| **Command** | `npx -y @modelcontextprotocol/server-filesystem ${WORKSPACE_ROOT}/src ${WORKSPACE_ROOT}/docs/architecture` |
| **Required env** | `WORKSPACE_ROOT` |
| **Scopes** | Read access to the scoped paths |

## Setup

1. Set `WORKSPACE_ROOT`; adjust the path arguments to match your repo layout (e.g.
   replace `src` with your solution folder, or add more roots).
2. No credentials required.

## Notes

- Credential-free and intentionally scoped (least privilege) versus the broader
  `code-search` server.
- Pair with the `documentation-search` server to validate patterns against guidance.
