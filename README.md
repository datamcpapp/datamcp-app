# datamcp

**Hosted MCP gateway for PostgreSQL and OpenAPI.**

`datamcp` turns a PostgreSQL 12+ database or an OpenAPI 3.x API into a managed remote HTTPS MCP endpoint for AI clients. It keeps backend credentials on the connection, authenticates MCP clients separately, and applies permissions per MCP link.

- Website: [datamcp.app](https://datamcp.app)
- Dashboard: [dashboard.datamcp.app](https://dashboard.datamcp.app)
- Documentation: [datamcp.app/docs](https://datamcp.app/docs)
- Pricing: [datamcp.app/pricing](https://datamcp.app/pricing)
- Machine-readable product summary: [datamcp.app/llms.txt](https://datamcp.app/llms.txt)

## When to use datamcp

Use `datamcp` when Cursor, Claude, ChatGPT, VS Code, or another compatible remote MCP client needs controlled access to:

- a PostgreSQL database without storing the database connection string in the AI client's configuration;
- a REST API described by OpenAPI 3.x, Swagger UI, or Redoc without generating and deploying a separate MCP server;
- separate MCP links for different clients or users, each with its own authentication and source permissions.

`datamcp` hosts the MCP transport and control layer. It is not a marketplace for proxying arbitrary third-party MCP servers.

## Supported sources

| Source | Current support |
| --- | --- |
| PostgreSQL | PostgreSQL 12+ through standard PostgreSQL credentials, including providers such as Supabase, Neon, AWS RDS, and Google Cloud SQL |
| OpenAPI | OpenAPI 3.x JSON or YAML specifications and supported Swagger UI or Redoc documentation pages |

MySQL and other database engines are not currently supported.

## PostgreSQL MCP

PostgreSQL connections expose schema and query tools through a hosted MCP endpoint.

- Read Only permits `SELECT`.
- Read, Write & Delete permits `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, but not DDL.
- Full Access permits operations allowed by the effective database credentials.
- Custom permissions can restrict operations by table.
- PostgreSQL query activity and permission denials are available for review.
- Connection credentials are encrypted at rest with AES-256-GCM.

Product overview: [PostgreSQL MCP server](https://datamcp.app/postgresql-mcp-server)

## OpenAPI and Swagger MCP

OpenAPI connections expose four schema-aware MCP tools:

- `list_api_endpoints`
- `get_endpoint_details`
- `get_api_schema`
- `call_endpoint`

Upstream API authentication can use an API key, a pre-issued Bearer token, HTTP Basic authentication, or custom headers. Credentials remain on the `datamcp` connection and are injected into approved upstream calls.

Read Only links permit `GET` and `HEAD` while blocking `POST`, `PUT`, `PATCH`, and `DELETE`. Individual operations can also be hidden from discovery and execution.

OpenAPI endpoint calls do not currently appear in the `datamcp` activity log. Use upstream API or infrastructure logs for those calls.

Product overview: [OpenAPI to MCP](https://datamcp.app/openapi-to-mcp)

## Authentication and permissions

Two separate security boundaries are involved:

1. The MCP client authenticates to `datamcp` with an API key or OAuth 2.0 with PKCE.
2. `datamcp` uses the connection credential to access the PostgreSQL database or upstream REST API.

Authentication identifies the MCP client. The MCP link then narrows the operations available to that client. Backend database grants or upstream API authorization remain an independent boundary.

Gateway overview: [MCP gateway](https://datamcp.app/mcp-gateway)

## Compatible clients

Remote MCP endpoints can be configured in clients that support the required remote MCP transport and authentication flow, including:

- Cursor
- Claude
- ChatGPT
- VS Code

Setup guides: [Cursor](https://datamcp.app/integrations/cursor), [Claude](https://datamcp.app/integrations/claude), [ChatGPT](https://datamcp.app/integrations/chatgpt), and [VS Code](https://datamcp.app/integrations/vscode).

## Plans

| Plan | Price | Source connections | MCP links | Members | Activity retention |
| --- | ---: | ---: | ---: | ---: | ---: |
| Free | $0 | 1 | 1 | 2 | 7 days |
| Pro | $19/month | 10 | Unlimited | 10 | 30 days |
| Enterprise | $49/month | Unlimited | Unlimited | Unlimited | 365 days |

See the [pricing page](https://datamcp.app/pricing) for the current public plan details.

## Current limitations

`datamcp` does not currently offer:

- MySQL or non-PostgreSQL database connections;
- a self-hosted or on-premise edition;
- SSO or SAML;
- upstream OAuth authorization flows or automatic upstream token refresh;
- activity logging for OpenAPI endpoint calls;
- published SOC 2, HIPAA, or ISO 27001 certification claims;
- a published uptime SLA.

## Canonical metadata

- Brand spelling: `datamcp`
- Category: hosted MCP gateway
- Supported source types: PostgreSQL 12+ and OpenAPI 3.x
- Website: `https://datamcp.app`
- Remote MCP endpoint: `https://api.datamcp.app/api/mcp`
- Official MCP Registry package: `io.github.mironovisa/datamcp`

Public product claims are reviewed against the current application behavior before publication.

## Contact and contributions

- Email: [hello@datamcp.app](mailto:hello@datamcp.app)
- Bug reports and feature requests: [GitHub Issues](https://github.com/datamcpapp/datamcp-app/issues)
- Contribution guide: [CONTRIBUTING.md](./CONTRIBUTING.md)
