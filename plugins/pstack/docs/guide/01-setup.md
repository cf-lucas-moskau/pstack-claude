# Set up pstack

In this page you install the plugin, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

In Claude Code, run:

```text
/plugin marketplace add cf-lucas-moskau/pstack-claude
/plugin install pstack@pstack-claude
```

Then restart Claude Code so the plugin's skills and agents load. pstack's skills are namespaced, so you type them as `/pstack:<skill>`.

## Pick your models

Run:

```text
/pstack:setup-pstack
```

[`/pstack:setup-pstack`](../../skills/setup-pstack/SKILL.md) detects the models you have access to, asks for a reasoning budget, shows you each role (code delegates, judgment, the review panels), and asks what you want. Answer the questions. It writes `~/.claude/pstack/models.md`, one line per role in the form `model@effort`, such as `sonnet@xhigh` or `opus@xhigh`. The models are `opus`, `sonnet`, `haiku`, `fable`, or `inherit`, and the effort runs `low`, `medium`, `high`, `xhigh`, `max`. Every pstack skill that spawns role-based subagents reads this file when it starts.

The defaults run at `xhigh` effort, the same as the `large` budget: `sonnet@xhigh` for code delegates and `opus@xhigh` for judgment. `unlimited` lifts every role to `max`. `medium` and `small` lower the effort to `high` and `medium` and spend fewer tokens.

You only override what you care about. A role with no line in the file keeps the skill's default. To restore a default, delete that role's line. A rerun of `/pstack:setup-pstack` keeps any role whose model differs from the default. When a default changes, a file written before the change still pins the old default, so delete those role lines, or delete the file, then run `/pstack:setup-pstack` again.

You might be wondering how to keep a role on the model you're already chatting with. Set it to `inherit` and pstack omits the subagent's `model` and `effort`, so the subagent inherits your main session's model. For a panel role the value is a list, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default model for every `/pstack:swarm` worker unless a race names a model for each arm.

## Accept the verification offer, or don't

At the end of setup, `/pstack:setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/pstack:create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.claude/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/pstack:create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to pstack, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

There's nothing to restart after setup. Each skill reads `~/.claude/pstack/models.md` when it runs, so the next pstack run uses your choices.

## Keep the cost in check

pstack spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `/pstack:setup-pstack` and pick a smaller reasoning budget or cheaper models. A strong model in the main chat with cheaper, faster models in the code roles is a good split.
- Set a role to `inherit` so it runs on the main session's model.
- Shorten a panel list. Each entry runs one subagent.
- Save `/pstack:poteto-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/pstack:poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/pstack:poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. Start each task with `/pstack:poteto-mode`; it stays on for that task. When you switch to a new task, start it with `/pstack:poteto-mode` again.

Next: [Route work through `/pstack:poteto-mode`](./02-poteto-mode.md).
