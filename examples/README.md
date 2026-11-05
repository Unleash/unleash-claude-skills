# Examples

Alternative configurations you can swap in for the defaults this skill ships with.

## `mcp.remote.json` — Remote HTTP transport

The default `.mcp.json` at the repo root runs the Unleash MCP server locally over stdio via `npx`. Use the remote HTTP transport instead when:

- Your org disallows running local MCP server processes (no `npx`, no Node.js on developer machines).
- You prefer a centrally hosted MCP server you control through Unleash admin settings.
- Your Unleash instance admin has enabled the remote MCP endpoint and your identity provider permits OAuth 2.0 Dynamic Client Registration.

### How to swap

Replace the contents of the root `.mcp.json` with `examples/mcp.remote.json`, then set the URL to your instance:

```bash
cp examples/mcp.remote.json .mcp.json
```

Then open `.mcp.json` and replace `https://your-instance.getunleash.io` with your actual Unleash host.

If you are configuring the server outside a plugin install (for example, a project-level `.mcp.json` or a Claude Desktop connector), use the same `type: http` block.

### Required setup on the Unleash side

1. Sign in to Unleash as an instance admin.
2. Go to **Admin settings → Remote MCP server**.
3. Toggle **Enable Remote MCP Server for this instance** to **Enabled**.
4. Confirm with your identity provider admin that OAuth 2.0 Dynamic Client Registration (DCR) is allowed for the Unleash app.

### Auth flow

You don't need a token. On first use, Claude Code opens your browser to log in to Unleash (run `/mcp` if it doesn't). Unleash then issues the client a personal access token that is valid for **24 hours**, and the server acts with your permissions.

Where a browser login isn't possible, such as in CI, send a personal access token of your own instead and no login happens:

```json
"headers": {
  "Authorization": "Bearer ${UNLEASH_PAT}"
}
```

Add this next to `url` in `.mcp.json`. The token keeps the expiry you set when you created it. See the [Unleash MCP integration docs](https://docs.getunleash.io/integrate/mcp) for the current auth flow.

### Trade-offs

| | Local stdio (default) | Remote HTTP |
|---|---|---|
| Node.js 24+ required | ✅ | — |
| Credentials | Your own PAT in `UNLEASH_PAT` | Browser login (or a PAT header for CI) |
| Admin must enable a server-side toggle | — | ✅ |
| OAuth DCR required | — | ✅ |
| Works against any Unleash instance | ✅ | Only instances with remote MCP enabled |
| Centralized server (no per-laptop process) | — | ✅ |

If you're unsure, start with the default local mode and switch later. The skills and their reference files work the same with both transports.
