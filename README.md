# Ashura

Ashura is a Claude Code plugin: the mini version of [Sonny Boy](https://github.com/rit1217/sonny-boy).
It is a head-of-team orchestrator on top of a small dev pipeline
(planner → tester → coder → advisor when blocked → reviewer), and it stays fully local:

- splits one or many requirements into subtasks (`decomposer`);
- gives each subtask its own git worktree, branch and lead agent (`lead`), in parallel;
- relays every question to you as a question box labelled `[Tn · title]`;
- asks you to approve acceptance criteria and the test design per subtask, then builds tests
  first (red → green);
- checks each subtask once with a fresh `reviewer`, merges it into a local run branch
  `ashura/<run>`, runs the full test suite once on that branch, and hands you that one
  **local branch** plus `pr-body.md`. You merge it. Ashura never touches your working branch,
  never pushes, never opens a PR.

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
| `/ashura:ashura status` | status board; detects when you merged the run branch |
| `/ashura:ashura pause` / `resume [run]` | pause safely; resume in a fresh session from files |

Agents are named `ashura:planner`, `ashura:coder`, and so on, so they never collide with agents
in `~/.claude/agents/`.

Per repo, Ashura writes run state to `<repo>/.claude/ashura/<run>/` and worktrees for the run
branch `ashura/<run>` and the subtask branches `ashura/<run>/<Tn>-<slug>`, and adds `.claude/ashura/` and `.claude/worktrees/` to
`.git/info/exclude`.

The first time you run Ashura, it asks where to put worktrees and saves the answer in
`~/.claude/ashura/config.md` (`worktree_root`). It applies to all repos. Choose a folder outside
OneDrive, Dropbox or iCloud: each subtask is a full checkout, and a synced folder makes checkouts
and test runs slow and can lock files on Windows. `worktree_root: repo` keeps the old layout,
`<repo>/.claude/worktrees/ashura-<run>-<Tn>`; an absolute path gives
`<worktree_root>/<repo folder name>/ashura-<run>-<Tn>`. Edit or delete the file to change it.
Repo-specific facts (python path, `tests_targeted`, `tests_full`) go in an optional
`<repo>/.claude/ashura/project.md`; template in `skills/ashura/references/templates.md`.

## Speed

A subtask costs four agents: planner, tester (writes the failing tests from the plan, before
any code), coder (code only, never touches those tests), reviewer (one independent check). Agents run only
targeted tests. The full suite runs once per batch, on the run branch, and every test run is
cached by commit SHA in `test-runs.tsv`, so the same commit is never tested twice. Fix loops stop
after 2 rounds; then you decide. Dependent subtasks start after their parent lands on the run
branch, so there are no stacked branches or restacks, and a break between two subtasks shows up
when the second one lands.

## Handoff

When every subtask has landed and the full suite passes, Ashura shows the run branch's diff stat
and commits, how each acceptance criterion was proved, the manual checks and the path of
`pr-body.md`, then prints `git merge --no-ff ashura/<run>`. It detects the merge locally with
`git merge-base --is-ancestor` and removes clean worktrees afterwards.

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
| `agents/` | 8 agents: lead, decomposer, planner, tester, coder, advisor, reviewer, integrator |

Models: Opus for planning and judgement (Ashura, lead, decomposer, planner, advisor, reviewer);
Sonnet for writing tests and code, and for merges (tester, coder, integrator).

## Credits

- **Name** — Ashura is named after the creator's favorite song, *修羅(Shura)* by ヨルシカ(Yorushika),
  without any special meaning.
- **[Sonny Boy](https://github.com/rit1217/sonny-boy)** — the parent plugin Ashura is cut down from.
- **[Caveman](https://github.com/JuliusBrussee/caveman)** by Julius Brussee — terse communication
  mode. Required plugin.
- **[Matt Pocock's skills](https://github.com/mattpocock/skills)** — grilling and domain-modeling,
  used by the planner. Required plugin.
- **[pstack](https://github.com/cursor/plugins/tree/main/pstack)** by Lauren Tan — the
  orchestration principles in `skills/ashura/references/principles.md`. Adapted, not copied.

## License

MIT. See [LICENSE](LICENSE).
