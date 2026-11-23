# Ashura templates

## state.md
```markdown
# Ashura run <run>
phase: setup | decompose | running | integrate | paused | closed
mode: nested | flat
working: <branch>
run_branch: ashura/<run>
run_wt: <abs path>
top: <abs path>

## Goal
<2 lines: goal restated; smallest change>

## Requirements
- R1 <≤6-word title>: <user's text verbatim>

## Environment
worktree_root: <repo | abs path>
os: <Darwin | Linux | WSL | Windows-GitBash>
python: <abs path>
tests_targeted: <command with {paths}, e.g. "<python> -m pytest -q {paths}">
tests_full: <whole-suite command; Ashura runs it once per batch only>
last_full: <sha> pass | fail | none
graphify: yes | no
document-update: yes | no

## House rules
- <rule copied from CLAUDE.md / memory>

## Subtasks
| T | title | R | branch | base | wt | lead id | status |
|---|---|---|---|---|---|---|---|

## Queue
<subtasks waiting for a lead slot or a parent>

## Question queue
<FIFO: Tn, question id, asked yes/no>
```
Subtask status ∈ queued, plan, red, green, check, READY_FOR_MERGE, LANDED (merged into the run
branch), HANDED_OFF, MERGED (run branch merged into working), paused, failed, dropped.

## decisions.tsv (header)
```
ts	phase	decision	why	evidence	result
```
Rows only for real decisions: lane, retry, escalation, plan change, landing, cleanup.

## test-runs.tsv (header; `<run_dir>/test-runs.tsv`, shared by all agents of the run)
```
sha	scope	command	result	ts
```
`scope` = `full` or `targeted:<test files>`. Same `sha` + same `scope` already present → reuse the
result, never re-run. Only rows from a clean tree count.

## config.md (per user, all repos: `~/.claude/ashura/config.md`)
```markdown
# Ashura user config
worktree_root: <repo | absolute path, forward slashes, e.g. C:/Users/me/ashura-worktrees>
```
`repo` = worktrees under `<top>/.claude/worktrees/`. Absolute path = worktrees under
`<worktree_root>/<repo folder name>/`. Edit or delete this file to change it; deleted → Ashura asks
again on the next run.

## project.md (optional, per repo: `<top>/.claude/ashura/project.md`)
```markdown
# Ashura project facts
python: <path relative to top or absolute, e.g. .venv/Scripts/python.exe>
tests_targeted: <command taking test paths, e.g. "<python> -m pytest -q {paths}">
tests_full: <whole suite, e.g. "<python> -m pytest -q">
frontend_dir: <e.g. frontend, or none>
frontend_tests: <e.g. "cd frontend && npm test", or none>
never_fast_lane: <globs that always get the full lane, e.g. src/pricing/**, or none>
notes: <repo-specific traps>
```

## Lead kickoff
```
You are ashura:lead for <Tn> "<title>" in run <run>.
mode: new | resume | fix
worktree: <wt abs path>
branch: <branch>
base_ref: ashura/<run>
task_dir: <task_dir abs path>
run_dir: <run_dir abs path>          (test-runs.tsv lives here)
state: <run_dir>/state.md            (read Environment + House rules)
subtask: <run_dir>/subtasks.md#<Tn>  (goal, done evidence, file boundary)
qa_log: <task_dir>/qa.md
templates: <this file's abs path>
Return only your STATUS block (first line STATUS: <TOKEN>, ≤15 lines; QUESTIONS: full round).
```

## Status board
```
**R1 <title>** — in progress · 1/3 subtasks done
<1 line: what has landed so far, what is left for this requirement>
- [x] T1 <title> — landed
- [ ] T2 <title> — check · waiting on reviewer
- [ ] T3 <title> — queued · waiting on T2

**R2 <title>** — ready to merge · 2/2 subtasks done
<1 line summary>
- [x] T4 <title> — landed
- [x] T5 <title> — landed

Overall: 0/2 requirements done · 3/5 subtasks landed on ashura/<run> · full suite: <sha> pass ·
1 question queued
```
Phase ∈ plan, red, green, check, landed, handed off, merged, paused, failed, dropped.

## PR body (written by Ashura at handoff to `<run_dir>/pr-body.md`, for the user to paste)
```markdown
## Why
<1–3 lines: problem and goal>

## What changed
- <bullet per meaningful change>

## Scope
Ashura run <run>, branch `ashura/<run>`. Subtasks: <Tn title — boundary, one line each>.
Out of scope: <items>.

## Tradeoffs
<what was chosen over what, and why>

## Blast radius
<callers / modules / users affected; risk level>

## Verification
- Full suite on <sha>: <command> → <N passed>
- <Tn>: <done evidence command> → <result>; AC1 → <test id / evidence>
- Manual: <manual check, if any>
```
≤60 lines. End with the PR attribution line from House rules.
