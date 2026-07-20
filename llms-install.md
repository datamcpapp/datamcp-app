# Install datamcp in Cline

`datamcp` is a hosted remote MCP gateway. Do not clone this repository or start a local process.

## Prerequisites

1. Create an account at https://dashboard.datamcp.app.
2. Add a PostgreSQL 12+, MySQL, or OpenAPI connection.
3. Create an MCP link with the required source permissions.
4. Copy the generated remote MCP URL and its client API key from the link setup guide.

The generated URL identifies the MCP link. The API key authenticates the MCP client. Do not use the generic product endpoint in place of the generated link URL.

## Cline CLI

Run Cline's interactive MCP manager:

```bash
cline mcp
```

Then select:

1. **Add server**
2. Server name: `datamcp`
3. Server type: **Remote (HTTP)**
4. Server URL: your generated MCP link URL
5. Authentication: **Static headers**
6. Headers: `Authorization:Bearer YOUR_CLIENT_API_KEY`

Use the complete generated MCP link URL and replace `YOUR_CLIENT_API_KEY` with the client API key shown in the link setup guide. Current Cline CLI releases manage MCP servers interactively; `cline mcp install` is not a supported command.

## Manual Cline configuration

Add the server to Cline's MCP settings using its Streamable HTTP transport:

```json
{
  "mcpServers": {
    "datamcp": {
      "type": "streamableHttp",
      "url": "https://api.datamcp.app/api/mcp/conn_REPLACE_ME",
      "headers": {
        "Authorization": "Bearer sk_live_REPLACE_ME"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

For Cline CLI, the global MCP settings file is `~/.cline/data/settings/cline_mcp_settings.json`. In the Cline editor extension, open **MCP Servers → Configure MCP Servers** instead of guessing the settings path.

## Verify the connection

After Cline reports the server as connected:

1. Ask it to list the available database schema or API endpoints.
2. Run one operation allowed by the MCP link.
3. Attempt an operation denied by the link policy and confirm that it is blocked.

For setup details and troubleshooting, use https://datamcp.app/docs. Never commit a real API key, MCP link secret, database connection string, or upstream API credential to a repository.
