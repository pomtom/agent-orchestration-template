# Governance: Security Policy

Security expectations enforced by the framework's standards, reviews, and scanning MCP
servers. Detailed coding rules live in the standards under
[`docs/standards/`](../standards/); this is the governing policy.

## Secrets

- **Never commit secrets** — no keys, passwords, connection strings, or tokens in
  source, `appsettings.json`, or `local.settings.json`.
- Local: **User Secrets**. Deployed: **Key Vault** / App Settings, preferably via
  **Managed Identity** (see [configuration-standard](../standards/configuration-standard.md)).
- MCP credentials are environment variables only; `.mcp/configurations/.env` is
  gitignored. Rotate tokens regularly and scope them minimally.

## Application security (assessed in every architecture review)

- Authentication and authorization cover every exposed endpoint/trigger.
- Validate and sanitize all external input; guard against injection and insecure
  deserialization.
- Enforce transport security (HTTPS/TLS); least-privilege RBAC on Azure resources.
- **No sensitive data in logs** — redact PII/secrets
  ([logging-standard](../standards/logging-standard.md)).
- Error responses leak no internals; clients get a generic message + correlation ID
  ([exception-handling-standard](../standards/exception-handling-standard.md)).

## Supply chain

- No vulnerable/deprecated packages; scanned on every PR
  ([package-governance](./package-governance.md)).
- `dependency-scanning` (SCA) and `security-scanning` (SAST/secrets) MCP servers feed the
  `tech-debt-assessment` workflow.

## Reviews & scanning

- Architecture reviews include a dedicated security dimension
  ([architecture-standard](../standards/architecture-standard.md)).
- CI runs vulnerable-package scanning (`standards-ci.yml`).

## Reporting a vulnerability

Report suspected vulnerabilities privately to the repository maintainers /
security contact (configure `SECURITY.md` in the host repo). Do not open a public issue
for an unpatched vulnerability.
