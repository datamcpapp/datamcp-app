# DataMCP

**Managed MCP server for PostgreSQL. Connect Cursor, Claude, and other AI tools to your database with granular permission control, query audit logs, and zero infrastructure to manage.**

DataMCP is a secure PostgreSQL MCP gateway that sits between your database and AI coding assistants. Instead of pasting raw credentials into Cursor or giving Claude full database access, you connect them through DataMCP - which enforces permissions at the query level, logs every request, and blocks dangerous operations before they reach your database.

Works with Cursor, Claude Desktop, VS Code, Windsurf, Claude Code, and any MCP-compatible client.

Website: [datamcp.app](https://datamcp.app) | Dashboard: [dashboard.datamcp.app](https://dashboard.datamcp.app) | Docs: [datamcp.app/docs](https://datamcp.app/docs)

---

## What it does

1. You paste your PostgreSQL connection string into DataMCP
2. We encrypt it (AES-256-GCM), extract your schema, and generate AI-readable descriptions for every table and column
3. You get an MCP URL that you drop into Cursor, Claude, or any MCP-compatible tool
4. AI sees your schema, can run queries within your permission rules, and helps you build faster

No SDK. No code changes. One URL, one API key — works in under 60 seconds.

---

## Core features

### MCP Tools (what AI gets)

| Tool | What it does |
|---|---|
| `query` | Execute SQL (SELECT, INSERT, UPDATE, DELETE — depending on permissions). 100-row limit, 30-second timeout. |
| `get_schema` | Full schema: tables, columns, types, foreign keys, indexes — with AI-generated descriptions merged in. |
| `get_table_details` | Deep dive into a specific table: columns, constraints, relationships. |
| `get_permissions` | What this MCP link is allowed to do. AI can self-check before attempting a query. |
| `get_schema_changes` | Diff history between schema versions. "What changed since last week?" |
| `resync_schema` | Re-extract schema from the live database when AI detects it's stale. |

### Permission system

Every MCP link has its own permission scope. You choose:

- **Read-only** — SELECT queries, view schema
- **Read-write** — SELECT + INSERT, UPDATE, DELETE
- **Full access** — Everything including DDL (CREATE, ALTER, DROP)
- **Custom** — Per-table control: allow SELECT on `users` but block `billing_events` entirely

Every query is validated against the permission scope **before** execution. Denied queries are logged and returned to the AI with an explanation of why they were blocked.

### AI schema descriptions

DataMCP auto-generates a one-sentence description for every table and a 6-word description for every column using OpenAI gpt-4o-mini. These descriptions are served to AI tools through `get_schema`, dramatically improving query quality on ambiguous schemas.

- Descriptions are stored in DataMCP's metadata layer — we never write `COMMENT ON` to your database, never require write access
- You can review and edit every description in the dashboard before it reaches your AI
- On schema changes (new tables/columns detected via resync), we generate descriptions only for the new items and mark them for review with a `NEW` badge
- Rule-based fallback for common patterns (created_at, user_id, etc.) if the LLM is unavailable

### Connection health monitoring

- Automated health checks every hour using the same SSL strategy as real queries
- 5 consecutive failures: connection marked as Error, MCP links disabled, email alert sent to org owners and admins
- Auto-recovery: next successful health check restores the connection automatically
- One-click **Reconnect** from the dashboard when your database comes back up
- Real PostgreSQL error messages shown directly on the connection card

### Live activity indicator

Every connection card shows how many unique AI clients (Cursor, Claude, etc.) and dashboard users ran a query in the last 15 minutes. Backed by Redis sorted sets — survives restarts, consistent across replicas. Hover for a breakdown of what's being counted and the timestamp of the most recent query.

### Activity logs

Every SQL query executed through MCP is logged:

- Query text, execution status, execution time (ms), row count
- Which MCP link and permission preset was used
- Permission violations: which rule blocked a denied query
- Retention: 7 days (Free), 30 days (Pro), 365 days (Enterprise)

### Organizations and team access

- Every connection, MCP link, and team member belongs to an organization
- Roles: **Owner** (full control), **Admin** (manage members + connections), **Member** (use connections, create own MCP links)
- Invite teammates by email, revoke anytime
- Per-organization billing and plan limits

---

## Supported AI tools

| Tool | Integration | Config |
|---|---|---|
| Cursor | Native MCP | `.cursor/mcp.json` with `url` + `Authorization` header |
| Claude Desktop | via mcp-remote | `claude_desktop_config.json` with `npx mcp-remote` |
| VS Code | Native MCP | `mcp.json` with `url` + `Authorization` header |
| Windsurf | Native MCP | Same as VS Code |
| Claude Code (CLI) | Native MCP | `claude mcp add --transport streamable-http` |
| Kiro | Native MCP | Same as VS Code |
| Zed | Native MCP | Same as VS Code |
| Any MCP client | MCP Protocol | Standard streamable-http transport |

## Supported databases

PostgreSQL 12+ from any provider:

Supabase, Neon, AWS RDS, Google Cloud SQL, Microsoft Azure, Heroku Postgres, DigitalOcean, and any self-hosted PostgreSQL accessible over the internet.

SSL/TLS verified with bundled CA certificates for all major cloud providers. Self-signed certificates supported via configurable SSL mode.

---

## Pricing

|  | Free | Pro | Enterprise |
|---|---|---|---|
| **Price** | $0 forever | $19/month | $49/month |
| PostgreSQL connections | 1 | 3 | 15 |
| MCP links per connection | 1 | 5 | 50 |
| Team members per org | 2 | 5 | 25 |
| Activity log retention | 7 days | 30 days | 365 days |
| Custom per-table permissions | — | Yes | Yes |
| Audit export | — | — | Yes |
| Priority support | — | — | Yes |

No contracts. No cancellation fees. Downgrade to Free anytime.

---

## Security

| Layer | Implementation |
|---|---|
| Credentials at rest | AES-256-GCM encryption. Plaintext connection strings never stored. |
| API key storage | SHA-256 hashed. Only the prefix stored for identification. |
| MCP client auth | OAuth 2.0 with PKCE, or API key via Bearer token. |
| Query validation | Every SQL statement parsed and checked against permission scope before execution. |
| Database connections | SSL/TLS always enabled. Verified CA certificates for cloud providers. |
| Audit trail | Every query logged with timestamp, user, execution time, and result metadata. |

---

## Architecture

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────┐
│  AI Tool     │────>│  DataMCP Gateway │────>│  Your        │
│  (Cursor,    │ MCP │  (NestJS)        │ SSL │  PostgreSQL  │
│   Claude,    │<────│                  │<────│              │
│   VS Code)   │     │  - Auth          │     └──────────────┘
└─────────────┘     │  - Permissions   │
                    │  - Query exec    │
                    │  - Schema cache  │
                    │  - AI descriptions│
                    │  - Audit logging │
                    └──────────────────┘
                           │
                    ┌──────┴───────┐
                    │  Dashboard   │
                    │  (Next.js)   │
                    │              │
                    │  - Connections│
                    │  - MCP links │
                    │  - Logs      │
                    │  - Billing   │
                    │  - Settings  │
                    └──────────────┘
```

**Backend:** NestJS, BullMQ (job queues), Redis (caching + live activity), Supabase (auth + data), Stripe (billing)

**Frontend:** Next.js 16 (App Router), React 19, Tailwind CSS 4, shadcn/ui, React Query

**Infrastructure:** Render.com, Supabase, Redis, Resend (transactional email), Stripe (payments)

---

## Market position

### What we are

A **middleware layer** between PostgreSQL databases and AI-powered development tools. We solve the problem of "how do I let Cursor/Claude see my database schema and run queries without giving it my raw connection string and full admin access."

### Who it's for

- **Individual developers** using AI coding assistants who want their AI to understand their database schema
- **Dev teams** who want to share database access across multiple AI tools with different permission levels per tool and per person
- **CTOs and security-conscious orgs** who need audit trails and per-table permission control before allowing AI tools to touch production data

### Competitive landscape

| Alternative | What DataMCP solves |
|---|---|
| Pasting schema into AI chat | Stale, manual, no query execution, no permission control |
| Direct DB connection in AI tool | No permission layer, no audit trail, credentials in plaintext config files |
| Building a custom MCP server | Weeks of engineering, no dashboard, no team features, no monitoring |
| Database GUI with AI features | Vendor lock-in to one tool; DataMCP works with any MCP client |

### Key differentiators

1. **Protocol-native.** Built on MCP from day one. Works with any MCP-compatible client automatically — present and future.
2. **Permission-first.** AI can't do anything you didn't explicitly allow. Not "AI has admin, we hope it behaves."
3. **Multi-tool.** One connection serves Cursor, Claude Desktop, VS Code, and any future MCP client simultaneously — each with its own permission scope.
4. **Zero-install for end users.** No SDK, no package, no code changes. One URL in a config file.
5. **AI-enhanced metadata.** Auto-generated schema descriptions make AI dramatically better at understanding ambiguous column names and table relationships.

---

## Current state (April 2026)

Live and serving production traffic. Users connect real PostgreSQL databases and query them through AI tools daily.

**Recent releases (v0.9.x):**

- AI schema descriptions with incremental generation on schema changes
- Connection error handling with one-click reconnect and email alerts
- Live activity indicator (Redis-backed, unique clients per 15-minute window)
- Schema resync from dashboard with real-time progress bar
- Email preferences with GDPR-compliant opt-in/out
- Persistent connection pooling (10x query speedup)
- SSL CA certificate bundles for all major cloud providers

**Coming next:**

- PostgreSQL read-replica proxy (direct psql/pgAdmin access through DataMCP's permission layer)
- AI chat interface in the dashboard
- Self-hosted edition (Docker, deploy anywhere)

---

## FAQ

**Is DataMCP a managed MCP server or self-hosted?**
Managed. You sign up, connect your database, and get an MCP URL. No server to run, no infrastructure to manage. A self-hosted Docker edition is on the roadmap.

**How is this different from just connecting Cursor directly to my database?**
Direct connection gives Cursor your full credentials with no restrictions - it can SELECT, DELETE, DROP, anything your database user can do. DataMCP adds a permission layer in front. You define exactly what queries are allowed, and every query goes through validation before it hits your database. Plus you get a full audit log of what the AI actually ran.

**How do I set up PostgreSQL permissions for AI tools like Cursor?**
The recommended approach with DataMCP: create a dedicated read-only PostgreSQL user for AI access, connect it to DataMCP, then use DataMCP's permission presets to further restrict access (specific schemas, specific tables, SELECT-only). This gives you two independent permission layers - PostgreSQL role-level and DataMCP application-level. DataMCP's effective permissions = the intersection of both.

**Can I give different AI tools different permission levels?**
Yes. Each MCP link has its own permission preset. You can give your local Cursor READ_WRITE on the dev schema, give a contractor's Claude READ_ONLY on a specific schema only, and block production data entirely for external tools - all from the same database connection.

**Does DataMCP work with Cursor specifically?**
Yes. Cursor supports MCP natively. You add DataMCP's URL to `.cursor/mcp.json` with an Authorization header. Takes about 2 minutes to set up.

**What PostgreSQL databases does it support?**
Any PostgreSQL 12+ instance reachable over the internet: AWS RDS, Supabase, Neon, Google Cloud SQL, Azure, Heroku, Railway, Render, DigitalOcean, or self-hosted.

**Is this secure for production databases?**
DataMCP is built specifically for this use case. Credentials are encrypted at rest with AES-256-GCM, never stored as plaintext. Every query is parsed and validated before execution. You can restrict access to read-only, specific tables, or specific schemas. TRUNCATE/DROP/ALTER are blocked by default. Full audit trail of every query the AI ran.

**How much does it cost?**
Free tier: 1 database, 1 MCP link. Pro: $19/month for 3 databases and 5 MCP links. Enterprise: $49/month for 15 databases and 50 MCP links.

---

## Links

- **Website:** [datamcp.app](https://datamcp.app)
- **Dashboard:** [dashboard.datamcp.app](https://dashboard.datamcp.app)
- **Documentation:** [datamcp.app/docs](https://datamcp.app/docs)
- **Changelog:** [datamcp.app/changelog](https://datamcp.app/changelog)
- **Pricing:** [datamcp.app/pricing](https://datamcp.app/pricing)
- **Contact:** hello@datamcp.app
