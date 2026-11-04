# Hooks (optional, opt-in)

This template provides two automation behaviors around feature-flag work. They live in different places:

| Behavior | How it works | Where it lives |
|---|---|---|
| **Flag audit** (on demand) | Ask Claude to "run a flag audit" and the skill follows the audit workflow | `skills/featureops/references/audit-cadence.md` → *Auditing* |
| **High-risk-edit check** (on file edit) | A `PostToolUse` hook matching `Edit`/`Write` that reminds you to evaluate risky changes for a flag | `evaluate-high-risk-save.example.json` (this directory) |

## Why the file-edit hook is opt-in

Claude Code has no dedicated "file save" event. The closest equivalent is a `PostToolUse` hook that fires after the `Edit` or `Write` tools, narrowed to a path with the `if` field.

Unlike a plugin's auto-discovered `hooks/hooks.json` (which activates whenever the plugin is enabled), this hook ships as an **example** file that is *not* loaded automatically — a deliberate default so nothing fires until you opt in. Hooks fire on real tool events and consume tokens, so they should be opt-in and scoped to your project.

## Enable it

Edit `evaluate-high-risk-save.example.json` first:

| Field | Default | Change to |
|---|---|---|
| `if` | `Edit(src/payments/**)` + `Write(src/payments/**)` | The glob for *your* high-risk directories, e.g. `Edit(src/auth/**)`. The example ships a paired `Edit` and `Write` entry because the `if` permission-rule syntax is per-tool — one entry only covers one tool, so both are needed to catch edits *and* new files. Add more entries for multiple domains. |
| `command` | Reminder to follow the featureops skill | Add domain-specific guidance if your high-risk areas use different naming patterns |

Then activate it one of two ways:

1. **Merge into settings** — copy the `hooks` block into your project `.claude/settings.json` (or `~/.claude/settings.json` for all projects).
2. **Activate as a plugin hook** — rename the file to `hooks/hooks.json` in your local copy of this repo. It will load whenever the `unleash` plugin is enabled. (Reverting to opt-in = rename it back.)

Reload with `/reload-plugins` (plugin hooks) or restart Claude Code (settings hooks), then confirm the hook appears in `/hooks`.

## Cost note

`PostToolUse` hooks fire on *every* matching edit. Keep the `if` glob narrow (`Edit(src/payments/checkout/**)` beats `Edit(src/**)`) and consider excluding test files. This example uses a lightweight `command` handler that only prints a reminder; if you switch to a `prompt` or `agent` handler, each fire runs an LLM turn and costs more.

## Related

- Conventions the hook references: `../skills/featureops/references/evaluate-change.md`, `../skills/featureops/references/high-risk-domains.md`
- Claude Code hooks reference: https://code.claude.com/docs/en/hooks
