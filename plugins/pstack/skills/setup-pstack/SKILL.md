---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes ~/.claude/pstack/models.md, which pstack's skills read to override their defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.claude/pstack/models.md`, a plain markdown file that sets pstack's model and effort per role. It is not auto-loaded. Every pstack skill that spawns role-based subagents reads it before its first spawn and passes each role's value as the Agent tool's `model` and `effort`.

## Steps

### 1. Detect available models

The available models are the values of the Agent tool's `model` parameter in this session: `opus`, `sonnet`, `haiku`, and `fable`. Read the enum from the tool schema and use what it actually lists. The effort levels are the Agent tool's `effort` values, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. The alias `inherit` is always valid even though it is not in the enum. It means the role runs on the parent chat's model and effort, so the skill omits both `model` and `effort`. Never write a model that is not in the detected set.

### 2. Load current state

The default role-to-model mapping is the file shape shown in step 5 below. If `~/.claude/pstack/models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

If `~/.claude/pstack/models.md` does not exist but `~/.cursor/rules/pstack-models.mdc` does, a Cursor install of pstack configured it before. Offer once, with `AskUserQuestion`, to migrate its choices. On yes, read its `# budget` line and role lines and map each value to a family: a Claude Opus slug becomes `opus`, a Claude Sonnet slug `sonnet`, a Claude Haiku slug `haiku`, and any fast code-model slug from another vendor `sonnet`. `inherit-parent` and `auto` become `inherit`. Keep the old budget label. Mark any slug you cannot map as needing a choice. On no, start from the defaults. Never edit or delete the old file.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use `AskUserQuestion` with these four options and these exact labels, and name the current budget when the file records one. With no file, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role whose model or list you changed, or that you set to `inherit`. The budget sets the `@effort` suffix on every role, panel entries included: `unlimited` → `max`, `large` → `xhigh`, `medium` → `high`, `small` → `medium`. `inherit` entries do not change. So `unlimited` turns `opus@xhigh` into `opus@max`, `large` keeps the defaults, and `small` turns `sonnet@xhigh` into `sonnet@medium`.

**(c) Show the roles and confirm.** Show every role with its value, marking any model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask with `AskUserQuestion` whether to accept as-is or change specific roles. For a change, offer the detected models plus `inherit` (this role runs on the parent chat's model) as options, two to four per question. `AskUserQuestion` adds a free-text answer, so a list or an explicit `model@effort` can come through it. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, `inherit` entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every model written must be in the detected set, and every effort must be one of the Agent tool's `effort` values. `inherit` always passes. If a chosen value is not available, stop and ask again.

### 5. Write the file

Create `~/.claude/pstack/` if needed and write `~/.claude/pstack/models.md` with no frontmatter: a `# budget` line with the chosen label and its effort, then one line per role as `role: model@effort`, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration. One line per role, as model@effort. Delete a line to fall back to the skill default.
# `inherit` as a value: the role runs on the parent chat's model and effort (omit Agent `model` and `effort`). `inherit` entries in a panel list still count toward its fan-out.
# budget: large (xhigh)
feature, refactoring: sonnet@xhigh
bug-fix: sonnet@xhigh
perf-issue: sonnet@xhigh
hillclimb: sonnet@xhigh
judgment and prose: opus@xhigh
hardest tasks: opus@xhigh
how explorer: sonnet@xhigh
how explainer: opus@xhigh
why investigators: sonnet@xhigh
why synthesizer: opus@xhigh
reflect tooling: sonnet@xhigh
reflect judgment, divergent, synthesizer: opus@xhigh
arena runners: opus@xhigh, sonnet@xhigh
arena cross-judge pool: opus@xhigh, sonnet@xhigh
swarm workers: sonnet@xhigh
architect runners: opus@xhigh, sonnet@xhigh
interrogate reviewers: opus@xhigh, sonnet@xhigh
```

### 6. Confirm

Tell the user the file was written. pstack's skills read it at each run, so it applies from the next task on, including in this session. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /pstack:create-verification-skill." On yes, invoke `/pstack:create-verification-skill`. On no, move on without pushing.
