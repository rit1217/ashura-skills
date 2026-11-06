# Ashura

Ashura is a Claude Code plugin: the mini version of [Sonny Boy](https://github.com/rit1217/sonny-boy).
It is a head-of-team orchestrator on top of a small dev pipeline
(planner → tester → coder → advisor → reviewer), and it stays fully local:

- splits one or many requirements into subtasks (`decomposer`);
- gives each subtask its own git worktree, branch and lead agent (`lead`), in parallel;
- relays every question to you as a question box labelled `[Tn · title]`;
- asks you to approve acceptance criteria and the test design per subtask, then builds tests
  first (red → green);
- runs a fresh `verifier` per subtask and hands you a verified **local branch** plus
  `pr-body.md`. You merge. Ashura never merges, never pushes, never opens a PR.

Differences from Sonny Boy: no GitHub CLI, no Playwright, no e2e lane. The test design levels
are `unit | integration | manual`; `manual` rows are listed as manual checks in `verify.md`.

## Install

In Claude Code:

```
/plugin marketplace add https://github.com/juliusbrussee/caveman.git
/plugin marketplace add https://github.com/rit1217/ashura-skills.git
/plugin install caveman@caveman
/plugin install mattpocock-skills@claude-plugins-official
/plugin install ashura@ashura-skills
```

`ashura` depends on `caveman@caveman` (terse output; every agent preloads it) and
`mattpocock-skills@claude-plugins-official` (the planner's grilling and domain-modeling skills).
Install both yourself; the plugin reports "failed to load" until they are present
(`/plugin list` shows the status). Start a new session after installing.

If you still have the user-scoped copies (`~/.claude/skills/ashura/`, `~/.claude/agents/ashura-*.md`),
remove them after installing, so that `/ashura` and the plugin do not both appear.

## Commands

| Command | What |
|---|---|
| `/ashura:ashura <requirements>` | new run: decompose, build in worktrees, hand off branches |
| `/ashura:ashura add <requirement>` | add a requirement to the open run |
| `/ashura:ashura status` | status board; restack children whose parent branch was merged |
| `/ashura:ashura pause` / `resume [run]` | pause safely; resume in a fresh session from files |

Agents are named `ashura:planner`, `ashura:coder`, and so on, so they never collide with agents
in `~/.claude/agents/`.

Per repo, Ashura writes run state to `<repo>/.claude/ashura/<run>/` and worktrees to
`<repo>/.claude/worktrees/ashura-<run>-<Tn>`, on branches `ashura/<run>/<Tn>-<slug>`, and adds
both directories to `.git/info/exclude`. Repo-specific facts (python path, test commands) go in an
optional `<repo>/.claude/ashura/project.md`; template in `skills/ashura/references/templates.md`.

## Handoff

For each verified subtask Ashura shows the diff stat, the commits, how each acceptance criterion
was proved, the manual checks and the path of `pr-body.md`, then prints
`git merge --no-ff <branch>` commands in dependency order. It detects merges locally with
`git merge-base --is-ancestor` and, after asking you, restacks child branches with a local
`git rebase` (no fetch).

## Permissions

Agents inherit the session's permission mode. For a run, switch to accept-edits mode
(Shift+Tab) before starting `/ashura:ashura`, and allowlist routine commands (git add/commit,
`git worktree add`, your test runner) with `/permissions` or the `update-config` skill.
A plugin cannot set modes or permission rules itself.

## Layout

| Path | What |
|---|---|
| `.claude-plugin/` | `plugin.json` (plugin + dependencies), `marketplace.json` (this repo as a marketplace) |
| `skills/ashura/` | orchestrator skill + `references/` (principles, templates) |
| `agents/` | 9 agents: lead, decomposer, planner, tester, coder, advisor, reviewer, verifier, integrator |

Models: Opus for planning and judgement (Ashura, lead, decomposer, planner, advisor, reviewer);
Sonnet for writing code or tests and for checks (coder, tester, verifier, integrator).

## Credits

- **[Sonny Boy](https://github.com/rit1217/sonny-boy)** — the parent plugin Ashura is cut down from.
- **[Caveman](https://github.com/JuliusBrussee/caveman)** by Julius Brussee — terse communication
  mode. Required plugin.
- **[Matt Pocock's skills](https://github.com/mattpocock/skills)** — grilling and domain-modeling,
  used by the planner. Required plugin.
- **[pstack](https://github.com/cursor/plugins/tree/main/pstack)** by Lauren Tan — the
  orchestration principles in `skills/ashura/references/principles.md`. Adapted, not copied.

## License

MIT. See [LICENSE](LICENSE).
