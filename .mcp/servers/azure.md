# MCP Server: Azure

**Purpose:** Query and manage Azure resources (App Service, Function Apps, Storage,
Key Vault, App Configuration, etc.) to inform deployment/config/monitoring analysis.

| | |
|---|---|
| **Key (`.mcp.json`)** | `azure` |
| **Command** | `npx -y @azure/mcp@latest server start` |
| **Required env** | `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID` |
| **Scopes** | RBAC on the target subscription (Reader for analysis; more only if mutating) |

## Setup

1. Create an Azure AD app registration / service principal, or use a Managed Identity /
   `az login` (Azure CLI credential) locally.
2. Grant least-privilege RBAC (Reader is enough for assessment).
3. Export the `AZURE_*` env vars (see `.mcp/configurations/.env.example`).

## Notes

- Prefer Managed Identity / `DefaultAzureCredential` over client secrets where possible.
- Read-only by default; do not grant write/delete unless a task requires it.
