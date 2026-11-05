# Setup

This plugin drives the **Unleash MCP server** — connect that once and every skill works. Follow these steps after installing.

## 1. Prerequisites

- An **Unleash instance** (Cloud or self-hosted) with API access.
- For the local (stdio) transport: **Node.js 24+** and a **personal access token (PAT)** with permission to create and manage flags. The remote transport needs neither: you log in through your browser instead.

## 2. Connect the MCP server

- **Plugin install (Claude Code):** the bundled `.mcp.json` registers the server automatically — you only need to set the environment variables in step 3.
- **Standalone / other agents:** register the server yourself under the key `unleash`:

  ```bash
  claude mcp add unleash -- npx -y @unleash/mcp@0.6.0-beta.5
  ```

  (or your agent's MCP-config equivalent). For the remote HTTP transport, see [`examples/mcp.remote.json`](examples/mcp.remote.json).

## 3. Set credentials

```bash
export UNLEASH_BASE_URL=https://your-instance.getunleash.io   # the instance root, without /api
export UNLEASH_PAT=your-personal-access-token
```

For a reusable setup, keep these in `~/.unleash/mcp.env` and source it from your shell profile — see the [README](README.md#connect-the-local-server).

## 4. Verify

Restart your agent, then ask: *"list my Unleash projects."* If it returns your projects, setup is complete. If the Unleash tools aren't available, re-check the environment variables and that the MCP server is connected (in Claude Code, run `/mcp`).
