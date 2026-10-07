# pstack for Claude Code

Cursor ships [pstack](https://github.com/cursor/plugins/tree/main/pstack) and [cursor-team-kit](https://github.com/cursor/plugins/tree/main/cursor-team-kit) as Cursor plugins only. This repo is a Claude Code marketplace that points at those folders in [cursor/plugins](https://github.com/cursor/plugins), so you always get the current upstream version. No code is copied here.

## Install

```
claude plugin marketplace add cf-lucas-moskau/pstack-claude
claude plugin install pstack@pstack-claude
claude plugin install pstack-team-kit@pstack-claude
```

Or inside Claude Code: `/plugin marketplace add cf-lucas-moskau/pstack-claude`, then `/plugin install pstack@pstack-claude`.

Restart Claude Code afterwards. Start with `/pstack:poteto-help`.

## Update

```
claude plugin marketplace update pstack-claude
claude plugin update pstack@pstack-claude
```

## How it works

The upstream plugins have a `.cursor-plugin/plugin.json` but no `.claude-plugin/plugin.json`. The marketplace entries use `strict: false` with a `git-subdir` source, so Claude Code loads `skills/` and `agents/` straight from the upstream folder. Cursor-only parts (team-kit `rules/`) are ignored.

Both plugins are MIT licensed by their authors (Lauren Tan, Eric Zakariasson / Cursor).
