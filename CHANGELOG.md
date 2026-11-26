# Changelog

All notable changes to the `unleash` plugin. Versions follow [Semantic Versioning](https://semver.org); `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` (`metadata.version`) and `.devin-plugin/plugin.json` are kept in sync.

## 2.1.0

### Added

- Devin support. `.devin-plugin/plugin.json` makes this repository a Devin plugin for Devin Cloud, the Devin CLI and Devin Desktop. It declares the local MCP server inline under the key `unleash` (so Devin doesn't also load `.mcp.json`) with the credentials `UNLEASH_BASE_URL` and `UNLEASH_PAT`, and it supplies the Devin marketplace card: descriptions, setup notes and `assets/logo.png`.
- README: a "Use with Devin" section covering the Devin marketplace, `devin plugins install`, and credentials.

### Changed

- Skill text works in both Claude Code and Devin: it names the tool prefix for each install (`mcp__unleash__*` for the Devin plugin and standalone installs, `mcp__plugin_unleash_mcp__*` for the Claude Code plugin), mentions `devin mcp list` next to `claude mcp list`, and says what to do when `$ARGUMENTS`, `${CLAUDE_SKILL_DIR}` or the policy injection arrive unexpanded.

## 2.0.0

### Breaking

- Rebuilt on the new Unleash MCP server (`@unleash/mcp` 0.6.x). The v1 tools (`evaluate_change`, `detect_flag`, `create_flag`, `set_flag_rollout`, and so on) are replaced by `search`, `inspect`, `release`, `experiment`, `kill_switch`, `manage_flag`, `evaluate`, `configure`, `audit` and `docs`. Setups that pinned the old server will no longer match the skills.
- `featureops` references reworked: the v1 guidance tools moved into reference files (`evaluate-change`, `find-existing-flag`, `wrap-change`, `remove-flag`, `flag-usage-patterns`); `cleanup-cadence.md` renamed to `audit-cadence.md`; `feature-flag-conventions.md` removed.
- Remote MCP setup uses OAuth login.

### Changed

- The MCP server is pinned to `@unleash/mcp@^0.6.0` instead of `latest` in `.mcp.json`, `SETUP.md`, the README and the migration skill.
- `migrate-gitlab-flags` provisioning mapped to the new MCP tools.
- The example high-risk-edit hook points at the `featureops` workflow.

## 1.1.0

Last release on the v1 MCP server (`@unleash/mcp` 0.5.x). Earlier history: see git tags and PRs.
