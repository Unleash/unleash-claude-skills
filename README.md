# Unleash · FeatureOps Template for Claude

A [Claude Agent Skill](https://code.claude.com/docs/en/skills) that brings feature flag management into Claude Code. It configures the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) and ships a skill plus reference files that encode [FeatureOps](https://featureops.io) practices — evaluating changes for risk, creating flags with consistent naming, wrapping code, managing rollouts, and cleaning up stale flags.

The defaults work out of the box, but feature management conventions vary widely across teams and industries. This skill is designed as a **customizable template**. Fork it, adapt the reference files to your team's domains and conventions, and roll it out.

## What's inside

| Path | Purpose |
|---|---|
| `skills/featureops/SKILL.md` | The skill entry point. Frontmatter (name, description, pre-approved read-only tools), setup check, reference map, core workflow, tool reference, customization guide. |
| `skills/featureops/references/feature-flag-conventions.md` | Universal naming, types, and lifecycle rules. |
| `skills/featureops/references/rollout-guidance.md` | Default rollout milestones, hold durations, halt conditions. |
| `skills/featureops/references/cleanup-cadence.md` | Thresholds and workflow for removing stale flags, plus the flag-audit workflow. |
| `skills/featureops/references/high-risk-domains.md` | Worked example (payments) + template for adding your own domains. |
| `.mcp.json` | Default MCP server config — local stdio transport via `npx`. Auto-registered when installed as a plugin. |
| `examples/mcp.remote.json` | Alternative MCP config — remote HTTP transport for Unleash-hosted MCP. |
| `hooks/evaluate-high-risk-save.example.json` | Optional, opt-in `PostToolUse` hook that evaluates risky edits. Not loaded automatically. |
| `hooks/README.md` | The two flag-automation behaviors (on-demand audit, on-edit check) and how to enable the file-edit hook. |
| `.claude-plugin/plugin.json` | Plugin manifest (bundles the skill, MCP server, and — if you opt in — hooks). |
| `.claude-plugin/marketplace.json` | Marketplace manifest so the repo installs as a Claude Code plugin. |
| `LICENSE` | Apache-2.0. |

## Install

### Prerequisites

- **Claude Code** — a recent version (plugin marketplaces and bundled MCP servers required for Option A).
- An **Unleash instance** (Cloud or self-hosted) with API access.
- A **personal access token (PAT)** with permissions to create and manage flags.
- For local MCP transport: **Node.js 18+**.
- For remote MCP transport: the Unleash instance admin must enable **Admin settings → Remote MCP server**, and your identity provider must allow OAuth 2.0 Dynamic Client Registration.

### Option A — As a Claude Code plugin (recommended)

Installing as a plugin auto-registers the Unleash MCP server from the bundled `.mcp.json`.

```text
/plugin marketplace add Unleash/unleash-claude-skill
/plugin install unleash@unleash-marketplace
```

Then set your environment variables (below) and restart Claude Code or use `/reload-plugins`.

### Option B — As a personal or project skill

Copy the skill directory into your skills folder:

```bash
# personal (all your projects)
cp -r skills/featureops ~/.claude/skills/

# or project-scoped (this repo only)
mkdir -p .claude/skills && cp -r skills/featureops .claude/skills/
```

Because a standalone skill can't bundle an MCP server, register it yourself. Use the server key `unleash` so the skill's pre-approved tool names match (the bundled `.mcp.json` keys the server `mcp` for the *plugin* install path, so add it explicitly here rather than copying that file):

```bash
claude mcp add unleash -- npx -y @unleash/mcp@latest --log-level error
```

This minimal form relies on the environment variables you export below. Unleash's official [Claude Code integration guide](https://docs.getunleash.io/integrate/claude-code) shows the full `claude mcp add` command (with `-e` env flags and `--scope`) if you prefer to pass credentials inline.

### Environment variables

Both transports use Claude Code's `.mcp.json` environment-variable expansion, which supports `${VAR}` and the default form `${VAR:-default}` (the config uses `${UNLEASH_DEFAULT_PROJECT:-default}` so the project falls back to `default` when the variable is unset). Set:

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

Add to `~/.zshrc` or `~/.bashrc`:

```bash
if [ -f ~/.unleash/mcp.env ]; then
  set -a; source ~/.unleash/mcp.env; set +a
fi
```

Restart the shell, then restart Claude Code. Verify the tools are available by asking Claude *"list my Unleash projects"* (this calls the pre-approved read-only `list_projects` tool).

**Remote mode only needs `UNLEASH_PAT` or the OAuth flow** (the URL is in the config directly), but setting all three is harmless and lets you switch transports later.

## Use it

Once the MCP server is connected, the skill activates automatically when you talk about feature flags, rollouts, kill-switches, or Unleash. Or when you invoke it directly with `/unleash:featureops`. Typical prompts:

- *"Should this checkout change be behind a feature flag?"* → runs `evaluate_change`
- *"Create a release flag for the new search ranking and wrap the endpoint."* → `detect_flag` → `create_flag` → `wrap_change`
- *"Roll out `checkout-stripe-integration` to 10% in staging."* → `set_flag_rollout` / `toggle_flag_environment`
- *"Run a flag audit on the default project."* → the audit workflow in `cleanup-cadence.md`

The five read-only tools are pre-approved. Write operations always prompt for confirmation.

## Customize for your team

This skill ships as a template. Edit these files to match your conventions:

1. **`references/high-risk-domains.md`** — the file you'll edit most. Replace the payments example with your team's actual high-risk areas.
2. **`references/feature-flag-conventions.md`** — naming convention, flag types in scope, what counts as high-risk.
3. **`references/rollout-guidance.md`** — default milestones, hold durations, halt thresholds.
4. **`references/cleanup-cadence.md`** — stale-flag thresholds.
5. **`hooks/evaluate-high-risk-save.example.json`** — the `if` glob should match your high-risk directories (opt-in; see `hooks/README.md`).
6. **`SKILL.md` frontmatter** — add domain-specific keywords to the `description` so the skill activates for your team's vocabulary.

Every customizable spot in the reference files is marked with an HTML comment: `<!-- CUSTOMIZE: ... -->`. Run `grep -rn CUSTOMIZE skills/` from the repository root to find them all.

## Compatibility

- **Claude Code** — primary target. Install as a plugin (auto-registers MCP + optional hooks) or as a personal/project skill (configure MCP manually).
- **Portability** — `SKILL.md` follows the [Agent Skills](https://agentskills.io) open standard, so the skill folder is portable to other Agent Skills surfaces. MCP-driven flag management requires a runtime with network access and MCP support; the skill degrades to written guidance where those aren't available.
- **Local stdio transport** — works against any Unleash Cloud or self-hosted instance with API access.
- **Remote HTTP transport** — requires the Unleash instance admin to enable remote MCP and your IDP to permit OAuth 2.0 DCR. See `examples/README.md`.

## License and policies

- **License:** [Apache-2.0](./LICENSE)
- **Privacy Policy:** https://www.getunleash.io/privacy-policy
- **Support:** support@getunleash.io · file an issue at https://github.com/Unleash/unleash-claude-skill/issues
- **Maintainer:** Unleash (Bricks Software AS)

### MCP server attribution

This skill configures the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) ([`@unleash/mcp`](https://www.npmjs.com/package/@unleash/mcp) on npm), Apache-2.0 licensed and maintained by Unleash.

## Related

- [Unleash MCP server](https://github.com/Unleash/unleash-mcp) — the underlying server this skill wraps.
- [Unleash documentation](https://docs.getunleash.io) — full reference documentation.
- [Claude Code skills](https://code.claude.com/docs/en/skills) — how skills work in Claude Code.
- [Agent Skills standard](https://agentskills.io) — the open standard this skill follows.
- [FeatureOps](https://featureops.io) — the broader practice this skill supports.
