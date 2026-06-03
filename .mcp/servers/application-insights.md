# MCP Server: Application Insights

**Purpose:** Query Application Insights / Azure Monitor telemetry — traces, requests,
exceptions, dependencies, and custom dimensions (including the `CorrelationId`) for
end-to-end diagnostics and monitoring-strategy analysis.

| | |
|---|---|
| **Key (`.mcp.json`)** | `application-insights` |
| **Command** | `npx -y @azure/mcp@latest server start --namespace monitor` |
| **Required env** | `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID`, `APPLICATIONINSIGHTS_RESOURCE_ID` |
| **Scopes** | `Monitoring Reader` on the App Insights / Log Analytics resource |

## Setup

1. Reuse the Azure service principal from the `azure` server.
2. Grant `Monitoring Reader` on the Application Insights resource.
3. Set `APPLICATIONINSIGHTS_RESOURCE_ID` to the resource's full ARM ID.

## Notes

- Ties directly to the logging + correlation-ID standards: query by `CorrelationId`
  custom dimension to trace a request across services.
- Read-only telemetry access; no write scope needed.
