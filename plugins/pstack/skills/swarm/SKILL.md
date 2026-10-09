---
name: swarm
description: "Fan out N parallel workers, drain them, and return one report. Use for /swarm, 'swarm this', or parallel coverage, races, gauntlets, and exploration."
disable-model-invocation: true
---

# Swarm

Fan out N parallel workers, each in its own git worktree. They may cover separate slices, race the same brief, or mix both. The parent waits, aggregates, and returns one report.

## Start

Open a todolist with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Aggregate
4. Report

## Phase A: Frame

1. State the done predicate and the artifact or report the swarm must return.
2. Choose the shape. Partition into slices, race N workers on identical briefs, or mix both. For a race or mixed shape, declare `first pass`, `rank all`, or `best-of` before spawning.
3. Set N from the user or derive it from the shape. N is total workers, not a concurrency limit.
4. Pick the worker model. If `~/.claude/pstack/models.md` exists, read it; its lines override the defaults here. Use its `swarm workers` line, a `model@effort` value. If the file or that line is missing, use `sonnet@xhigh`. For `inherit`, omit `model` and `effort` so the workers run on the parent model. If the `Agent` tool rejects the configured value, use the default and say so. For a model race, name each arm's `model@effort` up front.
5. Give each worker its own writable output when it writes. When workers verify or measure commits, each brief names the exact SHAs. A measurement brief also names the method (sample count, what one sample is, order). The worker records both in its result.

## Phase B: Fan out

Spawn all N workers in one message as parallel `Agent` calls with `subagent_type: "general-purpose"`, `isolation: "worktree"`, `run_in_background: true`, and the step 4 `model` and `effort`, left unset for `inherit`. Each worker gets its own git worktree, so writing workers never collide. Drop `isolation` for a read-only worker that touches nothing.

When a worker must start from a non-default branch, name it in the brief and have the worker check it out in its worktree first.

For a durable run that must outlive this session (overnight, or on a schedule), set it up as a Claude Code routine with `/schedule` instead.

Every brief stands alone. Include the goal, scope, exact slice or race arm, how to verify, and what to report. Reports use `PASS`, `ISSUES`, or `BLOCKED` with evidence. A worker that can prove a defect reports `ISSUES` and lists every issue it can prove, not only the first.

If a worker drops out, proceed with N-1 and note it.

## Phase C: Aggregate

Read the terminal results. Drop a result that does not record the SHAs and method its brief names, and respawn that worker once. After a second miss, record a gap. A gap does not count as a pass. For coverage, every required slice needs a result. For a race, apply the selection rule declared up front. Use first pass, rank all, or best-of. Do not paste raw worker dumps.

Keep a compact result table, one-line evidenced issues, and explicit gaps or dropouts.

## Phase D: Report

Return one consolidated in-chat report with the table, issue one-liners, gaps or dropouts, and the race rule when used.
