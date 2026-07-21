# datamcp security model

`datamcp` is a hosted Model Context Protocol gateway for PostgreSQL 12+, MySQL, and OpenAPI 3.x. This document describes the product's current security boundaries and the controls users should combine with them.

## Separate authentication boundaries

There are two independent credential boundaries:

1. An MCP client authenticates to `datamcp` with an API key or OAuth 2.0 with PKCE.
2. `datamcp` uses the credential stored on the source connection to access PostgreSQL, MySQL, or an upstream REST API.

The database password or upstream API credential remains server-side instead of being copied into every AI client's configuration. Client authentication does not replace database grants or upstream API authorization.

## Scoped MCP links

Each generated MCP link has its own client authentication and source-specific permission policy.

For PostgreSQL and MySQL connections:

- Read Only permits `SELECT`.
- Read, Write & Delete permits `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, but not DDL.
- Full Access remains bounded by the effective database account.
- Custom permissions can narrow operations by table.

For OpenAPI connections:

- Read Only permits `GET` and `HEAD` while blocking `POST`, `PUT`, `PATCH`, and `DELETE`.
- Individual operations can be hidden from discovery and execution.
- An allowed `GET` is not guaranteed to be side-effect-free or non-sensitive; API semantics and API-side authorization remain the API owner's responsibility.

## Credential storage

Database credentials are encrypted at rest with AES-256-GCM. Use dedicated least-privilege database accounts, require provider-appropriate TLS, and avoid owner, administrator, or `root` accounts.

For OpenAPI connections, `datamcp` can store an API key, a pre-issued Bearer token, HTTP Basic credentials, or custom headers and inject them into approved upstream calls. It does not currently perform upstream OAuth authorization flows or automatic upstream token refresh.

## Revocation

Delete an MCP link to revoke that client's access without rotating the source credential used by other links. Inactive links are excluded from authenticated lookup.

Rotate the source credential separately when the database or upstream API credential itself may be compromised.

## Activity review

PostgreSQL and MySQL query execution and denied operations use the database activity path. Activity can be reviewed in the product and downloaded as CSV or JSON.

OpenAPI endpoint calls do not currently appear in the `datamcp` activity log. Use upstream API gateway, application, or infrastructure logs for those calls.

## Deployment boundaries

`datamcp` is a hosted service. It does not currently offer a self-hosted or on-premise edition, SSO or SAML, a published uptime SLA, or published SOC 2, HIPAA, or ISO 27001 certification claims.

## Recommended rollout

1. Create a dedicated least-privilege source credential.
2. Start with a Read Only MCP link.
3. Test both an allowed action and a denied action.
4. Grant writes or additional tables only when the workflow requires them.
5. Review database activity and rotate or revoke credentials when access changes.

## Reporting a security issue

Report security issues privately to [hello@datamcp.app](mailto:hello@datamcp.app). Do not include production credentials, API keys, database passwords, or sensitive customer data in the initial report.
