# Unleash for Claude Code

[![Discover on skills.sh](https://skills.sh/b/Unleash/unleash-claude-skills)](https://skills.sh/Unleash/unleash-claude-skills)

A Claude Code **plugin** that brings [Unleash](https://www.getunleash.io) feature management into your editor — a growing set of skills backed by the [Unleash MCP server](https://github.com/Unleash/unleash-mcp), encoding [FeatureOps](https://featureops.io) practices: controlling software behavior at runtime.

Skills work out of the box. Some — like [`featureops`](skills/featureops/) — also ship as **customizable templates** you fork and adapt to your team's conventions; each skill's README says whether and how.

## Skills

All skills install together under the `unleash` plugin, are invoked as `/unleash:<skill>`, and share one Unleash MCP server connection.

| Skill | Invoke | What it does |
|---|---|---|
| **[`featureops`](skills/featureops/)** | `/unleash:featureops` | Evaluate a change for risk, create flags with consistent naming, wrap code behind flags, plan progressive rollouts, and clean up stale flags. |
| **[`migrate-gitlab-flags`](skills/migrate-gitlab-flags/)** | `/unleash:migrate-gitlab-flags <app-dir>` | Migrate a codebase from GitLab Feature Flags to an Unleash server: inventory, concept mapping, provisioning, SDK rewrite, behavior-parity verification. Manual invocation only — it never auto-triggers. |

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

> **Version pinning:** the bundled config runs `@unleash/mcp@latest` via `npx`, so every session resolves the newest release. If your org requires vetted or reproducible dependencies, pin a version in `.mcp.json` (e.g. `@unleash/mcp@0.4.1`) and update it deliberately.

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

### Option C — via the skills.sh CLI (any agent)

The skills are also installable with [Vercel's `skills` CLI](https://skills.sh), which targets Claude Code and 70+ other agents straight from this repo:

```bash
npx skills add Unleash/unleash-claude-skills                     # choose skills interactively
npx skills add Unleash/unleash-claude-skills --skill featureops  # a specific skill
```

This copies the skill *files* into your agent — it does **not** set up the Unleash MCP server. Add that yourself (see [Environment variables](#environment-variables-shared-by-all-skills) and the `claude mcp add unleash` command above) so the MCP-backed tools work. `migrate-gitlab-flags` can also run through its import-file or Admin-API paths without MCP.

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
- **[`migrate-gitlab-flags`](skills/migrate-gitlab-flags/README.md)** — GitLab→Unleash feature flags migration: inventory, mapping, provisioning, SDK rewrite, parity verification. During a migration its parity rules take precedence over `featureops` conventions; afterwards `featureops` governs the migrated flags.

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
| `skills/<name>/` | One directory per skill: `SKILL.md`, `references/`, `README.md`, and optionally `scripts/` and `evals/`. |
| `.mcp.json` | Bundled Unleash MCP server config (local stdio via `npx`), server key `mcp`. Auto-registered on plugin install. The key is deliberate: it produces the plugin tool prefix `mcp__plugin_unleash_mcp__*` that every skill's `allowed-tools` pre-approves — renaming it silently disables those pre-approvals. |
| `examples/mcp.remote.json` | Remote HTTP transport alternative. |
| `hooks/` | Optional, opt-in `PostToolUse` example hook + docs. Not loaded automatically. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
| `.claude-plugin/marketplace.json` | Marketplace manifest (`unleash-marketplace`). |

## Compatibility

- **Claude Code** — primary target. Install as a plugin (auto-registers MCP + optional hooks) or copy individual skills (configure MCP manually).
- **Portability** — each `SKILL.md` follows the [Agent Skills](https://agentskills.io) open standard, so a skill folder is portable to other Agent Skills surfaces. MCP-driven features need a runtime with network + MCP support; skills degrade to written guidance where those aren't available.
- **Local stdio transport** — works against any Unleash Cloud or self-hosted instance with API access.
- **Remote HTTP transport** — requires the instance admin to enable remote MCP and your IDP to permit OAuth 2.0 DCR. See `examples/README.md`.

## Security & permissions

- **Bundled software:** the only external software this plugin configures is the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) ([`@unleash/mcp`](https://www.npmjs.com/package/@unleash/mcp), Apache-2.0, maintained by Unleash). No other binaries or network calls are added. Review the source before installing.
- **Credentials:** the server authenticates with an Unleash **personal access token** you provide via environment variables (`UNLEASH_PAT`); nothing is committed to the repo, and no credentials are bundled.
- **Tool approvals:** only the five **read-only** tools (`evaluate_change`, `detect_flag`, `get_flag_state`, `list_projects`, `list_flags`) are pre-approved. Every **write** operation (`create_flag`, `set_flag_rollout`, `toggle_flag_environment`, `remove_flag_strategy`) — and the code-editing helpers `wrap_change`/`cleanup_flag` — always prompt for explicit confirmation.
- **Scope of writes:** writes go only to the Unleash instance your token authorizes, at the permission level that token grants.

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
