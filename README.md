# Unleash for Claude Code

A Claude Code **plugin** that brings [Unleash](https://www.getunleash.io) feature management into your editor — a growing set of skills backed by the [Unleash MCP server](https://github.com/Unleash/unleash-mcp), encoding [FeatureOps](https://featureops.io) practices: controlling software behavior at runtime.

Skills work out of the box. Some — like [`featureops`](skills/featureops/) — also ship as **customizable templates** you fork and adapt to your team's conventions; each skill's README says whether and how.

The same plugin also installs in Devin: see [Use with Devin](#use-with-devin).

## Skills

All skills install together under the `unleash` plugin, are invoked as `/unleash:<skill>`, and share one Unleash MCP server connection.

| Skill | Invoke | What it does |
|---|---|---|
| **[`featureops`](skills/featureops/)** | `/unleash:featureops` | Evaluate a change for risk, create flags with consistent naming, wrap code behind flags, plan progressive rollouts, and clean up stale flags. |
| **[`migrate-gitlab-flags`](skills/migrate-gitlab-flags/)** | `/unleash:migrate-gitlab-flags <app-dir>` | Migrate a codebase from GitLab Feature Flags to an Unleash server: inventory, concept mapping, provisioning, SDK rewrite, behavior-parity verification. Manual invocation only — it never auto-triggers. |

More skills are on the way — see [Add a new skill](#add-a-new-skill).

## Install

Installing has two parts: the skills, and the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) they call. The server comes in two forms, remote and local, and the one you use decides how you sign in to Unleash.

### Prerequisites

- **Claude Code**, a recent version (plugin marketplaces and bundled MCP servers are required for Option A).
- An **Unleash instance** (Cloud or self-hosted), plus what the server you choose below needs.

### Choose the MCP server: remote or local

| | Remote MCP server | Local MCP server |
|---|---|---|
| Where it runs | On your Unleash instance, at `/api/admin/mcp` | On your machine, started through `npx` |
| How you sign in | Browser login (OAuth). Unleash issues a token valid for 24 hours | A personal access token (PAT) you create and set in `UNLEASH_PAT` |
| Which instances | Unleash Enterprise, with the remote MCP server turned on | Any instance, Cloud or self-hosted, including open source |
| Your machine needs | Nothing beyond Claude Code | Node.js 24+ |
| How to install the skills | Option B or C | Option A, B or C |

**Use the remote server if your instance offers it.** You don't create or store a token: you log in through your browser, and the token Unleash issues expires after 24 hours. It needs two things set up once for the instance:

- an instance admin turns on **Admin settings → Remote MCP server**;
- your identity provider allows OAuth 2.0 Dynamic Client Registration (DCR) for Unleash.

**Use the local server otherwise**, such as on an open-source instance or when the remote server is off. It authenticates only with a PAT that has permission to create and manage flags.

### Install the skills

#### Option A — Install the plugin (local server)

Installs the whole `unleash` plugin (every skill) and registers the **local** MCP server from the bundled `.mcp.json`. To use the remote server, install the skills with Option B or C instead.

```text
/plugin marketplace add Unleash/unleash-claude-skills
/plugin install unleash@unleash-marketplace
```

Then set the environment variables in [Connect the local server](#connect-the-local-server) and restart Claude Code or run `/reload-plugins`.

> **Server version:** the bundled config runs `@unleash/mcp@^0.6.0` via `npx`, so you get 0.6.x patch releases automatically but not a new minor version, which may change the tools. The skills call its tools (`search`, `inspect`, `release`, `manage_flag`, and so on), so if you change the range in `.mcp.json` (or `.devin-plugin/plugin.json` for Devin), check that the server still provides them.

#### Option B — Install a single skill standalone (either server)

Copy just the skill directory you want into your skills folder:

```bash
# personal (all your projects)
cp -r skills/featureops ~/.claude/skills/

# or project-scoped (this repo only)
mkdir -p .claude/skills && cp -r skills/featureops .claude/skills/
```

A standalone skill can't bundle an MCP server, so connect one yourself under the key `unleash`: see [Connect the remote server](#connect-the-remote-server) or [Connect the local server](#connect-the-local-server). The key `unleash` matches the skills' pre-approved tool names; the bundled `.mcp.json` keys the server `mcp` for the *plugin* path, so don't copy that file.

#### Option C — via the skills.sh CLI (any agent, either server)

The skills are also installable with [Vercel's `skills` CLI](https://skills.sh), which targets Claude Code and 70+ other agents straight from this repo:

```bash
npx skills add Unleash/unleash-claude-skills                     # choose skills interactively
npx skills add Unleash/unleash-claude-skills --skill featureops  # a specific skill
```

This copies the skill *files* into your agent; it does **not** set up the Unleash MCP server. Connect one yourself as in Option B so the MCP-backed tools work. `migrate-gitlab-flags` can also run through its import-file or Admin-API paths without MCP.

### Connect the remote server

Register the server with your instance's URL:

```bash
claude mcp add unleash https://your-instance.getunleash.io/api/admin/mcp --transport http
```

Add `--scope user` to make it available in all your projects. No environment variables are needed: on first use, Claude Code opens your browser to log in to Unleash (run `/mcp` if it doesn't), and the server acts with your permissions.

Where a browser login isn't possible, such as in CI, the remote server also accepts a PAT of your own as a header. See [`examples/README.md`](examples/README.md), which also covers swapping the plugin's `.mcp.json` for [`examples/mcp.remote.json`](examples/mcp.remote.json) when you run the plugin from a clone of this repo.

### Connect the local server

The local server reads two environment variables. Set:

```bash
export UNLEASH_BASE_URL=https://your-instance.getunleash.io   # the instance root, without /api
export UNLEASH_PAT=your-personal-access-token
```

A convenient pattern is a credentials file sourced from your shell profile:

```bash
mkdir -p ~/.unleash
cat > ~/.unleash/mcp.env << 'EOF'
UNLEASH_BASE_URL=https://your-instance.getunleash.io
UNLEASH_PAT=your-personal-access-token
EOF
chmod 600 ~/.unleash/mcp.env
```

Then in `~/.zshrc` or `~/.bashrc`:

```bash
if [ -f ~/.unleash/mcp.env ]; then
  set -a; source ~/.unleash/mcp.env; set +a
fi
```

Restart the shell. With the plugin (Option A), that's all: the bundled `.mcp.json` reads these variables. With a standalone install (Option B or C), also register the server:

```bash
claude mcp add unleash -- npx -y '@unleash/mcp@^0.6.0'
```

Unleash's official [Claude Code integration guide](https://docs.getunleash.io/integrate/claude-code) shows the full command (with `-e` env flags and `--scope`) if you prefer to pass credentials inline.

### Verify

Restart Claude Code and run `/mcp` to check that the Unleash server is connected. Then ask Claude *"list my Unleash projects"*.

## Use with Devin

[Devin](https://devin.ai) installs this repository as a plugin too. It reads [`.devin-plugin/plugin.json`](.devin-plugin/plugin.json), which loads both skills (`/unleash:featureops` and `/unleash:migrate-gitlab-flags`) and registers the **local** MCP server under the name `unleash`, so its tools are `mcp__unleash__*`. The manifest declares the server inline, which stops Devin from also loading the root `.mcp.json`, so the server isn't registered twice.

The server runs through `npx` and needs Node.js 24 or later where Devin runs it.

### Devin Cloud

1. In **Customize → Plugins**, search the **Marketplace** for **Unleash** and install it. To install straight from this repository instead, choose **Add plugin → From repository** and enter `Unleash/unleash-claude-skills`.
2. In **Customize → MCPs**, find the plugin's `unleash` server and choose **Connect**. Enter `UNLEASH_BASE_URL` (your instance root, without `/api`) and `UNLEASH_PAT` (a personal access token that can create and manage flags). Devin saves them on your installation; nothing is stored in this repository.
3. Start a new session and ask Devin to *"list my Unleash projects"*.

### Devin CLI and Devin Desktop

```bash
devin plugins install Unleash/unleash-claude-skills
```

This adds the plugin to your personal plugins, so it also loads in your Devin Cloud sessions and on your other devices. Add `--local` to install it on this machine only. Devin Desktop runs the same local agent as the CLI, so it loads the plugin too.

Locally, the server reads the same two environment variables as [Connect the local server](#connect-the-local-server). Set them in the shell you start Devin from (`devin`, or `devin desktop` for Desktop):

```bash
export UNLEASH_BASE_URL=https://your-instance.getunleash.io   # the instance root, without /api
export UNLEASH_PAT=your-personal-access-token
```

Run `devin mcp list` to check that `unleash` is listed, then ask Devin to *"list my Unleash projects"*.

The Devin CLI also imports MCP servers from Claude Code and Cursor configs, so a server you registered there as `unleash` has the same name as the plugin's. If `devin mcp list` shows an `unleash` entry whose source isn't the plugin, remove the duplicate, or turn off that import with `read_config_from` in `~/.config/devin/config.json`.

## Using the skills

Once installed, a skill activates automatically when relevant, or you invoke it directly as `/unleash:<skill>`. Read-only MCP tools are pre-approved while a skill is active; write operations always prompt for confirmation.

Each skill documents its own workflow, prompts, and customization in its README:

- **[`featureops`](skills/featureops/README.md)** — feature-flag lifecycle: risk evaluation, creation, wrapping, rollout, cleanup.
- **[`migrate-gitlab-flags`](skills/migrate-gitlab-flags/README.md)** — GitLab→Unleash feature flags migration: inventory, mapping, provisioning, SDK rewrite, parity verification. During a migration its parity rules take precedence over `featureops` conventions; afterwards `featureops` governs the migrated flags.

## Add a new skill

The plugin auto-discovers every skill under `skills/` (the marketplace entry scans `./skills/`), so adding one needs no manifest edits:

1. Create `skills/<name>/SKILL.md` with frontmatter `name: <name>` — it becomes `/unleash:<name>`. (Names can't contain "claude"/"anthropic".)
2. Add any `references/*.md` files and, for human docs, a `skills/<name>/README.md` (use `skills/featureops/` as the template).
3. Pre-approve read-only MCP tools in the skill's `allowed-tools` using **both** prefixes — `mcp__unleash__<tool>` (standalone installs and the Devin plugin) and `mcp__plugin_unleash_mcp__<tool>` (the Claude Code plugin). Never pre-approve write tools. Also add `Read(/${CLAUDE_SKILL_DIR}/**)` so the skill can read its own references without a prompt; the leading `/` makes the rule absolute. Don't inline files from `${CLAUDE_SKILL_DIR}` with `` !`cat …` ``: Claude Code only lets an injected `cat` read the session's working directories, and an installed plugin sits outside them, so the skill fails to load. Skills also load in Devin, which doesn't document Claude Code's `$ARGUMENTS`, `${CLAUDE_SKILL_DIR}` or `` !`…` `` injection: if you use them, say in prose what to do when they arrive unexpanded.
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
| `.mcp.json` | Bundled Unleash MCP server config for Claude Code (local stdio via `npx`), server key `mcp`. Auto-registered on plugin install. The key is deliberate: it produces the plugin tool prefix `mcp__plugin_unleash_mcp__*` that every skill's `allowed-tools` pre-approves — renaming it silently disables those pre-approvals. Devin doesn't load it. |
| `examples/mcp.remote.json` | Remote HTTP transport alternative. |
| `hooks/` | Optional, opt-in `PostToolUse` example hook + docs. Not loaded automatically. |
| `.claude-plugin/plugin.json` | Claude Code plugin manifest. |
| `.claude-plugin/marketplace.json` | Claude Code marketplace manifest (`unleash-marketplace`). |
| `.devin-plugin/plugin.json` | Devin plugin manifest: the Devin marketplace card (name, descriptions, setup notes, logo) and the `unleash` MCP server, declared inline. |
| `assets/logo.png` | Logo for the Devin marketplace card. |

## Compatibility

- **Claude Code** — primary target. Install as a plugin (auto-registers MCP + optional hooks) or copy individual skills (configure MCP manually).
- **Devin** — Devin Cloud, the Devin CLI and Devin Desktop install the plugin from the Devin marketplace or this repository, and it registers the local MCP server. See [Use with Devin](#use-with-devin).
- **Portability** — each `SKILL.md` follows the [Agent Skills](https://agentskills.io) open standard, so a skill folder is portable to other Agent Skills surfaces. MCP-driven features need a runtime with network + MCP support; skills degrade to written guidance where those aren't available.
- **Local stdio transport** — works against any Unleash Cloud or self-hosted instance with API access.
- **Remote HTTP transport** — requires the instance admin to enable remote MCP and your IDP to permit OAuth 2.0 DCR. See `examples/README.md`.

## Security & permissions

- **Bundled software:** the only external software this plugin configures is the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) ([`@unleash/mcp`](https://www.npmjs.com/package/@unleash/mcp), Apache-2.0, maintained by Unleash). No other binaries or network calls are added. Review the source before installing.
- **Credentials:** the local server authenticates with an Unleash **personal access token** you provide in `UNLEASH_PAT`. The remote server uses a browser login that issues a 24-hour token. In Devin Cloud, the token is saved on your plugin installation when you connect the server. Nothing is committed to the repo, and no credentials are bundled.
- **Tool approvals:** only **read-only** tools are pre-approved (`search`, `inspect`, `evaluate`, `audit`, `docs`, as each skill lists them). Every **write** tool (`release`, `kill_switch`, `experiment`, `manage_flag`, `configure`) always prompts for explicit confirmation.
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
