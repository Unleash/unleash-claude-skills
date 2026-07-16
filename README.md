# Unleash for Claude Code

A Claude Code **plugin** that brings [Unleash](https://www.getunleash.io) feature management into your editor — a growing set of skills backed by the [Unleash MCP server](https://github.com/Unleash/unleash-mcp), encoding [FeatureOps](https://featureops.io) practices: controlling software behavior at runtime.

Every skill ships as a **customizable template**. The defaults work out of the box; fork the repo and adapt each skill's reference files to your team's domains and conventions.

## Skills

All skills install together under the `unleash` plugin, are invoked as `/unleash:<skill>`, and share one Unleash MCP server connection.

| Skill | Invoke | What it does |
|---|---|---|
| **[`featureops`](skills/featureops/)** | `/unleash:featureops` | Evaluate a change for risk, create flags with consistent naming, wrap code behind flags, plan progressive rollouts, and clean up stale flags. |

More skills are on the way — see [Add a new skill](#add-a-new-skill).

## Install

### Prerequisites

- **Claude Code** — a recent version (plugin marketplaces and bundled MCP servers are required for Option A).
- An **Unleash instance** (Cloud or self-hosted) with API access.
- A **personal access token (PAT)** with permissions to create and manage flags.
- For local MCP transport: **Node.js 18+**.
- For remote MCP transport: the Unleash instance admin must enable **Admin settings → Remote MCP server**, and your identity provider must allow OAuth 2.0 Dynamic Client Registration.

### Option A — Install the plugin (recommended)

Installs the whole `unleash` plugin (every skill) and auto-registers the Unleash MCP server from the bundled `.mcp.json`.

```text
/plugin marketplace add Unleash/unleash-claude-skills
/plugin install unleash@unleash-marketplace
```

Then set your environment variables (below) and restart Claude Code or run `/reload-plugins`.

### Option B — Install a single skill standalone

Copy just the skill directory you want into your skills folder:

```bash
# personal (all your projects)
cp -r skills/featureops ~/.claude/skills/

# or project-scoped (this repo only)
mkdir -p .claude/skills && cp -r skills/featureops .claude/skills/
```

A standalone skill can't bundle an MCP server, so register it yourself under the key `unleash` (which matches the skills' pre-approved tool names — the bundled `.mcp.json` keys the server `mcp` for the *plugin* path, so add it explicitly here rather than copying that file):

```bash
claude mcp add unleash -- npx -y @unleash/mcp@latest --log-level error
```

Unleash's official [Claude Code integration guide](https://docs.getunleash.io/integrate/claude-code) shows the full command (with `-e` env flags and `--scope`) if you prefer to pass credentials inline.

### Environment variables (shared by all skills)

Claude Code's `.mcp.json` expansion supports `${VAR}` and the default form `${VAR:-default}` (the config uses `${UNLEASH_DEFAULT_PROJECT:-default}` so the project falls back to `default` when unset). Set:

```bash
export UNLEASH_BASE_URL=https://your-instance.getunleash.io/api
export UNLEASH_PAT=your-personal-access-token
export UNLEASH_DEFAULT_PROJECT=default   # optional; defaults to "default"
```

A convenient pattern is a credentials file sourced from your shell profile:

```bash
mkdir -p ~/.unleash
cat > ~/.unleash/mcp.env << 'EOF'
UNLEASH_BASE_URL=https://your-instance.getunleash.io/api
UNLEASH_PAT=your-personal-access-token
UNLEASH_DEFAULT_PROJECT=default
EOF
chmod 600 ~/.unleash/mcp.env
```

Then in `~/.zshrc` or `~/.bashrc`:

```bash
if [ -f ~/.unleash/mcp.env ]; then
  set -a; source ~/.unleash/mcp.env; set +a
fi
```

Restart the shell, then restart Claude Code. Verify by asking Claude *"list my Unleash projects"*.

**Remote mode only needs `UNLEASH_PAT` or the OAuth flow** (the URL is in the config directly); setting all three is harmless and lets you switch transports later. See `examples/README.md` for the remote transport.

## Using the skills

Once installed, a skill activates automatically when relevant, or you invoke it directly as `/unleash:<skill>`. Read-only MCP tools are pre-approved while a skill is active; write operations always prompt for confirmation.

Each skill documents its own workflow, prompts, and customization in its README:

- **[`featureops`](skills/featureops/README.md)** — feature-flag lifecycle: risk evaluation, creation, wrapping, rollout, cleanup.

## Customize for your team

Every skill ships as a template. Customizable spots in the reference files are marked with `<!-- CUSTOMIZE: ... -->`; find them all with:

```bash
grep -rn CUSTOMIZE skills/
```

See each skill's README for what to adapt (for example, [FeatureOps customization](skills/featureops/README.md#customize-for-your-team)).

## Add a new skill

The plugin auto-discovers every skill under `skills/` (the marketplace entry scans `./skills/`), so adding one needs no manifest edits:

1. Create `skills/<name>/SKILL.md` with frontmatter `name: <name>` — it becomes `/unleash:<name>`. (Names can't contain "claude"/"anthropic".)
2. Add any `references/*.md` files and, for human docs, a `skills/<name>/README.md` (use `skills/featureops/` as the template).
3. Pre-approve read-only MCP tools in the skill's `allowed-tools` using **both** prefixes — `mcp__unleash__<tool>` (standalone) and `mcp__plugin_unleash_mcp__<tool>` (plugin). Never pre-approve write tools.
4. Validate and test:
   ```bash
   claude plugin validate .
   claude --plugin-dir .
   ```
5. Add a row to the [Skills](#skills) table above.

## Repository layout

| Path | Purpose |
|---|---|
| `skills/<name>/` | One directory per skill: `SKILL.md`, `references/`, `README.md`. |
| `.mcp.json` | Bundled Unleash MCP server config (local stdio via `npx`), server key `mcp`. Auto-registered on plugin install. |
| `examples/mcp.remote.json` | Remote HTTP transport alternative. |
| `hooks/` | Optional, opt-in `PostToolUse` example hook + docs. Not loaded automatically. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
| `.claude-plugin/marketplace.json` | Marketplace manifest (`unleash-marketplace`). |

## Compatibility

- **Claude Code** — primary target. Install as a plugin (auto-registers MCP + optional hooks) or copy individual skills (configure MCP manually).
- **Portability** — each `SKILL.md` follows the [Agent Skills](https://agentskills.io) open standard, so a skill folder is portable to other Agent Skills surfaces. MCP-driven features need a runtime with network + MCP support; skills degrade to written guidance where those aren't available.
- **Local stdio transport** — works against any Unleash Cloud or self-hosted instance with API access.
- **Remote HTTP transport** — requires the instance admin to enable remote MCP and your IDP to permit OAuth 2.0 DCR. See `examples/README.md`.

## License and policies

- **License:** [Apache-2.0](./LICENSE)
- **Privacy Policy:** https://www.getunleash.io/privacy-policy
- **Support:** support@getunleash.io · file an issue at https://github.com/Unleash/unleash-claude-skills/issues
- **Maintainer:** Unleash (Bricks Software AS)

### MCP server attribution

This plugin configures the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) ([`@unleash/mcp`](https://www.npmjs.com/package/@unleash/mcp) on npm), Apache-2.0 licensed and maintained by Unleash.

## Related

- [Unleash MCP server](https://github.com/Unleash/unleash-mcp) — the underlying server these skills wrap.
- [Unleash documentation](https://docs.getunleash.io) — full reference documentation.
- [Claude Code skills](https://code.claude.com/docs/en/skills) · [plugins](https://code.claude.com/docs/en/plugins) — how skills and plugins work.
- [Agent Skills standard](https://agentskills.io) — the open standard each skill follows.
- [FeatureOps](https://featureops.io) — the broader practice these skills support.
