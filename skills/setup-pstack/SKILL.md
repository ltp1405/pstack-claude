---
name: setup-pstack
description: Configure which models pstack uses per role and at what budget. Writes an always-loaded Claude Code rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.claude/rules/pstack-models.md`, a user-level Claude Code rule that sets pstack's model per role. Claude Code loads every file in `~/.claude/rules/` without `paths` frontmatter into every session.

## Steps

### 1. Detect available models

The Agent tool's `model` parameter lists the values you can pass in this session. In Claude Code these are the aliases `opus`, `fable`, `sonnet`, and `haiku`. Never write a value the parameter does not accept. The alias `inherit-parent` is always valid: it means omit `model`, so the subagent runs on the parent chat model.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use AskUserQuestion. Offer these four options with these exact labels, and name the current budget when the rule records one.

- `unlimited — strongest models everywhere`
- `large — the defaults`
- `medium — sonnet for code and panels`
- `small — haiku for code, sonnet for judgment`

**(b) Apply it.** Build the working table from the step 5 defaults, and on a re-run keep any role the user changed by hand. Then apply the budget:

- `unlimited`: every code role and `swarm workers` become `opus`. Panels stay `opus, fable, sonnet`.
- `large`: the step 5 defaults unchanged.
- `medium`: code roles and `swarm workers` stay `sonnet`, judgment roles stay `opus`, panels become `opus, sonnet, sonnet`.
- `small`: code roles, `how explorer`, `why investigators`, and `swarm workers` become `haiku`. Judgment roles become `sonnet`. Panels become `sonnet, haiku, haiku`.

Panels are `arena runners`, `arena cross-judge pool`, `architect runners`, and `interrogate reviewers`. Judgment roles are `judgment and prose`, `hardest tasks`, `how explainer`, `why synthesizer`, and the `reflect` lines.

**(c) Show the roles and confirm.** Show every role with its model. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering `opus`, `fable`, `sonnet`, `haiku`, and `inherit-parent`. Use AskUserQuestion. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it that differs from the parent's model when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every value written must be one the Agent tool's `model` parameter accepts, or `inherit-parent`. If a chosen value is not valid, stop and ask again.

### 5. Write the rule

Write `~/.claude/rules/pstack-models.md` with a `# budget` line with the chosen label, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. No frontmatter: a `paths` key would scope the rule to matching files. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` as a value: the role runs on the parent chat model (omit Agent `model`). Alias entries in a panel list still count toward its fan-out.
# budget: large
feature, refactoring: sonnet
bug-fix: sonnet
perf-issue: sonnet
hillclimb: sonnet
judgment and prose: opus
hardest tasks: opus
how explorer: sonnet
how explainer: opus
why investigators: sonnet
why synthesizer: opus
reflect tooling: fable
reflect judgment, divergent, synthesizer: opus
arena runners: opus, fable, sonnet
arena cross-judge pool: opus, fable, sonnet
swarm workers: sonnet
architect runners: opus, fable, sonnet
interrogate reviewers: opus, fable, sonnet
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill`. On no, move on without pushing.
