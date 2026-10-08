---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Locate the active transcript

The parent finds its own transcript file before fanning out. Claude Code stores one JSONL per session under `~/.claude/projects/<encoded-cwd>/`, where `<encoded-cwd>` is the absolute working directory with every non-alphanumeric character replaced by `-`. Use only the current project's directory. Do not glob across `~/.claude/projects/*/`. That crosses project boundaries and reads private chats from unrelated projects.

```bash
ls -t ~/.claude/projects/<encoded-cwd>/*.jsonl 2>/dev/null | head -10
```

Sessions are `<session-id>.jsonl`; subagent transcripts sit under `<session-id>/subagents/`. The active session is usually the newest file.

For each candidate, check that its first `"type":"user"` line contains the conversation's opening user prompt (`grep -l` on a distinctive phrase works). Take the matching path. If no path resolves, write a tight digest of the session and pass that instead.

### 2. Spawn three reviewers in parallel

One message, three `Agent` calls, `subagent_type: "general-purpose"`, with `model` and `effort` set as below. Reviewers keep full tool access: they need MCPs for context lookups (tickets, chat threads, observability traces referenced in the transcript).

Each reviewer and the synthesizer name a role line and a default, both written `model@effort`. If `~/.claude/pstack/models.md` exists, read it; its lines override the defaults below. Pass the value's `model` and `effort`, or the default's if the file or the line is missing. Omit both when the value is `inherit`. If the `Agent` tool rejects a configured value, use the default and say so.

| Lens | Role line | Default | Prompt template |
|---|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `opus@xhigh` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `sonnet@xhigh` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `opus@xhigh` | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in their final report.

### 3. Synthesize

One `Agent` call, `subagent_type: "general-purpose"`, with `model` and `effort` from the `reflect judgment, divergent, synthesizer` line (default `opus@xhigh`), full tool access. The synthesizer's quality check includes spot-verifying citations, which can require MCP access. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. See the **encode-lessons-in-structure** principle skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): if the `skill-creator` skill is installed, run its draft / test / iterate loop. Otherwise edit the SKILL.md directly and keep its Agent Skills frontmatter (`name`, `description`) valid.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): use `skill-creator`'s description-optimization loop when installed; otherwise rewrite the `description` to name the triggers that were missed.
- `new skill: <kebab-name>`: write `<kebab-name>/SKILL.md` in Agent Skills format (frontmatter `name` matching the folder, `description`; optional `disable-model-invocation`, `allowed-tools`, `paths`), via `skill-creator` when installed. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done. Skip this step if it doesn't.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
