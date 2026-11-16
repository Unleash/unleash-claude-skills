# Changelog

All notable changes to the `unleash` plugin. Versions follow [Semantic Versioning](https://semver.org); `plugin.json` and `marketplace.json` (`metadata.version`) are kept in sync.

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
