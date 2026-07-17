# FeatureOps — `/unleash:featureops`

A skill in the [Unleash plugin for Claude Code](https://github.com/Unleash/unleash-claude-skills). It brings [FeatureOps](https://featureops.io) — the discipline of controlling software behavior at runtime — into Claude Code: evaluate changes for risk, create flags with consistent naming, wrap code behind flags, plan progressive rollouts, and clean up stale flags.

This skill ships as a **customizable template**: the defaults work out of the box, but you adapt the reference files to your team's domains and conventions — see [Customize for your team](#customize-for-your-team).

Install it with the plugin (`/plugin install unleash@unleash-marketplace`) or standalone (`cp -r skills/featureops ~/.claude/skills/`). See the [main README](https://github.com/Unleash/unleash-claude-skills#install) for MCP setup and environment variables.

## What's inside

| Path | Purpose |
|---|---|
| `SKILL.md` | Skill entry point: setup check, reference map, core workflow, tool reference. |
| `references/feature-flag-conventions.md` | Naming, flag types, and lifecycle rules. |
| `references/rollout-guidance.md` | Default rollout milestones, hold durations, halt conditions. |
| `references/cleanup-cadence.md` | Stale-flag thresholds, the removal workflow, and the flag-audit workflow. |
| `references/high-risk-domains.md` | Worked example (payments) + template for adding your own high-risk domains. |

## Use it

The skill activates automatically when you talk about feature flags, rollouts, kill-switches, experiments, or Unleash — or invoke it directly with `/unleash:featureops`. Typical prompts:

- *"Should this checkout change be behind a feature flag?"* → runs `evaluate_change`
- *"Create a release flag for the new search ranking and wrap the endpoint."* → `detect_flag` → `create_flag` → `wrap_change`
- *"Roll out `checkout-stripe-integration` to 10% in staging."* → `set_flag_rollout` / `toggle_flag_environment`
- *"Run a flag audit on the default project."* → the audit workflow in `references/cleanup-cadence.md`

The five read-only tools (`evaluate_change`, `detect_flag`, `get_flag_state`, `list_projects`, `list_flags`) are pre-approved while the skill is active; write operations always prompt for confirmation.

## Customize for your team

Edit these reference files to match your conventions — every customizable spot is marked with `<!-- CUSTOMIZE: ... -->`:

1. **`references/high-risk-domains.md`** — the file you'll edit most. Replace the payments example with your team's actual high-risk areas.
2. **`references/feature-flag-conventions.md`** — naming convention, flag types in scope, what counts as high-risk.
3. **`references/rollout-guidance.md`** — default milestones, hold durations, halt thresholds.
4. **`references/cleanup-cadence.md`** — stale-flag thresholds.
5. **`SKILL.md` frontmatter** — add domain-specific keywords to the `description` so the skill activates for your team's vocabulary.
6. **`hooks/evaluate-high-risk-save.example.json`** (in the plugin repository, not this skill directory) — the opt-in high-risk-edit hook; set its `if` glob to your high-risk directories (see [`hooks/README.md`](https://github.com/Unleash/unleash-claude-skills/blob/main/hooks/README.md)).

Find every customizable spot in this skill:

```bash
grep -rn CUSTOMIZE skills/featureops/
```
