# pstack for Claude Code

This repo is a Claude Code marketplace with two plugins:

- `pstack`: rigorous agent workflows, playbooks and principles (`/pstack:poteto-mode` and friends).
- `pstack-team-kit`: team workflows for CI, code review, shipping, UI and CLI verification, and work summaries.

Both are adapted from [cursor/plugins](https://github.com/cursor/plugins) (MIT). The code lives here and is rewritten for Claude Code: subagents use the `Agent` tool, questions use `AskUserQuestion`, and nothing depends on Cursor.

## Install

```
claude plugin marketplace add cf-lucas-moskau/pstack-claude
claude plugin install pstack@pstack-claude
claude plugin install pstack-team-kit@pstack-claude
```

Or inside Claude Code: `/plugin marketplace add cf-lucas-moskau/pstack-claude`, then `/plugin install pstack@pstack-claude`.

Restart Claude Code. Then:

1. Run `/pstack:setup-pstack` to choose a model and effort for each role. It writes `~/.claude/pstack/models.md`.
2. Start a task with `/pstack:poteto-mode <goal>. done when <check>.`
3. Lost? Run `/pstack:poteto-help`.

## Update

```
claude plugin marketplace update pstack-claude
claude plugin update pstack@pstack-claude
claude plugin update pstack-team-kit@pstack-claude
```

## What changed from the Cursor version

- Plugin manifests are `.claude-plugin/plugin.json`. The marketplace points at `./plugins/*` in this repo.
- Model roles use Claude Code models (`opus`, `sonnet`, `haiku`, `fable`, or `inherit`) plus an effort level, stored in `~/.claude/pstack/models.md`. Skills read that file when they spawn subagents.
- Cursor cloud agents became parallel `Agent` calls in their own git worktrees. Durable cloud runs use `/schedule` routines.
- Custom Modes do not exist in Claude Code. Start each task with `/pstack:poteto-mode`.
- The team-kit's always-on rules (`no-inline-imports`, `typescript-exhaustive-switch`) became skills.
- Removed because they only work in Cursor: the `make-bot-ui` skill (Grok Bot webhooks) and the `benny` Slack automations.

## Upstream sync

The vendored copy started from `cursor/plugins` commit `ccb5507` (2026-10-07). To pull upstream changes, diff that folder against a newer commit and port the changes by hand.

Both plugins are MIT licensed by their authors (Lauren Tan, Eric Zakariasson / Cursor).
