---
name: featureops
description: Manage feature flags with Unleash — evaluate whether a code change needs a flag, create flags with consistent naming, wrap code behind flags, plan progressive rollouts, and clean up stale flags. Use when working with feature flags, feature toggles, rollouts, releases, experiments, kill-switches, canary deploys, or the Unleash MCP server. Ships as a customizable FeatureOps template — adapt the reference files to your team's conventions.
license: Apache-2.0
allowed-tools: mcp__unleash__evaluate_change mcp__unleash__detect_flag mcp__unleash__get_flag_state mcp__unleash__list_projects mcp__unleash__list_flags mcp__plugin_unleash_mcp__evaluate_change mcp__plugin_unleash_mcp__detect_flag mcp__plugin_unleash_mcp__get_flag_state mcp__plugin_unleash_mcp__list_projects mcp__plugin_unleash_mcp__list_flags
---

# Unleash · FeatureOps Template

This skill brings [FeatureOps](https://featureops.io) — the discipline of controlling software behavior at runtime — into Claude Code. It drives the [Unleash MCP server](https://github.com/Unleash/unleash-mcp) to evaluate changes for risk, create flags with consistent naming, generate SDK guard code, manage rollouts, and clean up stale flags.

The defaults work out of the box, but feature management conventions vary across teams and industries. This skill ships as a **customizable template**: adapt the reference files under `references/` to your team's domains and conventions, then commit and share it.

## Setup check

This skill needs the Unleash MCP server connected. When installed as a Claude Code plugin, the server is registered automatically from the bundled `.mcp.json`; you only need to set three environment variables in the shell that launches Claude Code (or use the remote MCP server — see `examples/README.md` for details):

```bash
export UNLEASH_BASE_URL=https://your-instance.getunleash.io/api
export UNLEASH_PAT=your-personal-access-token
export UNLEASH_DEFAULT_PROJECT=default   # optional; defaults to "default"
```

If the Unleash tools are not available, tell the user to complete setup (see the repository README for full install, transport, and credential instructions) and stop rather than guessing flag state.

> **Tool naming note:** the read-only tools are pre-approved via `allowed-tools` under two names each: `mcp__unleash__*` (standalone-skill install, where you register the server under the key `unleash`) and `mcp__plugin_unleash_mcp__*` (plugin install, where Claude Code namespaces the bundled server — keyed `mcp` in `.mcp.json` — as `mcp__plugin_<plugin>_<server>__<tool>` → `mcp__plugin_unleash_mcp__<tool>`, verified on a real install). Pre-approval only applies while this skill is active; a read-only tool invoked before the skill has loaded will still prompt once. Write operations (`create_flag`, `set_flag_rollout`, `toggle_flag_environment`, `remove_flag_strategy`, `cleanup_flag`) are **never** pre-approved — always let the user confirm them.

## When to load the reference files

Load the reference file that matches the task. For tasks that span workflows (e.g. "create a flag and plan its rollout"), load both.

| User intent | Load |
|---|---|
| Setting up feature flags, deciding when to flag a change, naming a new flag | `references/feature-flag-conventions.md` |
| Working in a high-risk directory (payments, auth, data migrations, external integrations) | `references/high-risk-domains.md` |
| Planning a gradual rollout, configuring percentages, setting halt conditions | `references/rollout-guidance.md` |
| Removing a flag, cleaning up after a rollout, auditing stale flags | `references/cleanup-cadence.md` |

## Core workflow

For a change that might need a flag, follow this sequence. The reference files hold the detailed rules for each step.

1. **Evaluate risk** — call `evaluate_change` with a description of the change before implementing anything risky (payments, auth, data migrations, external integrations, performance-sensitive paths, user-visible behavior changes). See `references/feature-flag-conventions.md` and `references/high-risk-domains.md`.
2. **Prefer reuse** — call `detect_flag` before creating anything. Reuse a matching flag rather than adding to flag sprawl.
3. **Create** — if no match exists, call `create_flag` with a name following the team's convention (default `{domain}-{feature}-{variant}`) and the right type (`release`, `experiment`, `operational`, `kill-switch`, `permission`).
4. **Wrap** — call `wrap_change` for SDK-appropriate guard code. Apply it verbatim; keep the fallback branch realistic.
5. **Roll out** — use `set_flag_rollout` and `toggle_flag_environment` to progress through milestones. See `references/rollout-guidance.md`.
6. **Clean up** — flags are temporary by design. When a rollout stabilizes or an experiment concludes, remove the flag. See `references/cleanup-cadence.md`.

Do not auto-execute write operations. Present the plan, get confirmation, then act — especially for flag creation, rollout changes, and cleanup, which touch production code paths.

## Tool reference

The Unleash MCP server exposes the following tools.

| Tool | Description | When to use |
|---|---|---|
| `evaluate_change` | Analyzes a code change and determines whether it should be behind a feature flag. | Before implementing risky changes |
| `detect_flag` | Searches for existing flags that match a description to prevent duplicates. | Before creating new flags |
| `create_flag` | Creates a new feature flag with proper naming, typing, and metadata. | When no suitable flag exists |
| `wrap_change` | Generates framework-specific code to guard a feature behind a flag. | After creating a flag |
| `list_projects` | Lists Unleash projects available to the configured token, with optional pagination. | Discovering available projects |
| `list_flags` | Lists feature flags in a project (active by default; set `archived=true` for archived flags). | Auditing flag inventory; discovering existing flags before creating new ones |
| `get_flag_state` | Returns the current state, strategies, and metadata for a flag. | Debugging, status checks |
| `set_flag_rollout` | Configures rollout percentages and activation strategies. | Gradual releases |
| `toggle_flag_environment` | Enables or disables a flag in a specific environment. | Testing, staged rollouts |
| `remove_flag_strategy` | Deletes a rollout strategy from a flag. | Simplifying flag configuration |
| `cleanup_flag` | Returns file locations and instructions for removing a flag after rollout. | After full rollout |

The read-only tools — `evaluate_change`, `detect_flag`, `get_flag_state`, `list_projects`, `list_flags` — are pre-approved via `allowed-tools`. Write operations always require explicit user approval; keep it that way unless the user has a specific reason to lower the bar. Use `list_projects` and `list_flags` for inventory discovery before creating new flags; `list_flags` (active + archived) is also the basis of the flag audit workflow in `references/cleanup-cadence.md`.

## Customize this skill for your team

This skill ships as a template. Edit the reference files before rolling it out broadly:

- **High-risk domains** in `references/high-risk-domains.md` — replace the payments example with your own domains (auth, billing, data migrations, public API, etc.). *This is the file you'll edit most.*
- **Naming convention** in `references/feature-flag-conventions.md` — the default is `{domain}-{feature}-{variant}`; swap for your team's pattern.
- **Rollout milestones** in `references/rollout-guidance.md` — default is 10% → 50% → 100% with 24–48h holds (24h at 10%, 48h at 50%); adjust for your release cadence.
- **Cleanup cadence** in `references/cleanup-cadence.md` — default staleness signals are Unleash's `stale` marker, 30 days with no SDK evaluation, and Unleash's per-type lifetimes (release/experiment 40 days, operational 7 days); adjust for your tolerance.
- **Description keywords** in this file's frontmatter — add domain-specific vocabulary (e.g. "checkout flag", "billing rollout") so the skill activates for your team's terms.

Every customizable spot in the reference files is marked with an HTML comment: `<!-- CUSTOMIZE: ... -->`. From this skill's directory, run `grep -rn CUSTOMIZE references/` to find them all (from the repository root, use `grep -rn CUSTOMIZE skills/`).

---

## Metadata

- **License:** Apache-2.0
- **Privacy Policy:** https://www.getunleash.io/privacy-policy
- **Support:** support@getunleash.io · GitHub Issues at https://github.com/Unleash/unleash-claude-skill/issues
- **Source:** https://github.com/Unleash/unleash-claude-skill

### MCP server attribution

This skill configures the **Unleash MCP server**:

- Package: [`@unleash/mcp`](https://www.npmjs.com/package/@unleash/mcp) on npm
- Source: https://github.com/Unleash/unleash-mcp
- License: Apache-2.0
- Maintainer: Unleash (Bricks Software AS)
- Transport: local stdio via `npx`, or remote Streamable HTTP at `https://<your-instance>/api/admin/mcp`
