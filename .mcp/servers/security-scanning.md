# MCP Server: Security Scanning

**Purpose:** Application security scanning — SAST, secret detection, and dependency risk
— to feed the security dimension of architecture reviews and the tech-debt workflow.

| | |
|---|---|
| **Key (`.mcp.json`)** | `security-scanning` |
| **Command** | `npx -y @aikido/mcp-server` (placeholder — swap for your org's scanner) |
| **Required env** | `SECURITY_SCANNER_TOKEN` (maps to the vendor's token var) |
| **Scopes** | Read access to scan results for the target project/org |

## Setup

1. Choose your organization's security MCP server (e.g. Snyk, Trivy, Aikido, Microsoft
   Defender for Cloud) and update the `command`/`args` in `.mcp.json` accordingly.
2. Provision a scoped API token and export it as `SECURITY_SCANNER_TOKEN` (rename the
   env mapping in `.mcp.json` to the vendor's expected variable).

## Notes

- This entry is a **template**: the exact package and token var depend on your vendor.
- Pair with `dependency-scanning` (SCA) and the architecture review's security checks.
- Never commit the token.
