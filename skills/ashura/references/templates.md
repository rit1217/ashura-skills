# Ashura templates

## state.md
```markdown
# Ashura run <run>
phase: setup | decompose | running | integrate | paused | closed
mode: nested | flat
working: <branch>
top: <abs path>

## Goal
<2 lines: goal restated; smallest change>

## Environment
os: <Darwin | Linux | WSL | Windows-GitBash>
python: <abs path>
tests: <command(s)>
graphify: yes | no
document-update: yes | no

## House rules
- <rule copied from CLAUDE.md / memory>

## Subtasks
| T | title | branch | base | wt | lead id | status |
|---|---|---|---|---|---|---|

## Queue
<subtasks waiting for a lead slot or a parent>

## Question queue
<FIFO: Tn, question id, asked yes/no>
```
Subtask status ∈ queued, plan, red, green, verify, review, READY_FOR_MERGE, VERIFIED, HANDED_OFF,
MERGED, paused, failed, dropped.

## decisions.tsv (header)
```
ts	phase	decision	why	evidence	result
```

## project.md (optional, per repo: `<top>/.claude/ashura/project.md`)
```markdown
# Ashura project facts
python: <path relative to top or absolute, e.g. .venv/Scripts/python.exe>
tests: <unit/integration command, e.g. "<python> -m pytest -q">
frontend_dir: <e.g. frontend, or none>
frontend_tests: <e.g. "cd frontend && npm test", or none>
never_fast_lane: <globs that always get the full lane, e.g. src/pricing/**, or none>
notes: <repo-specific traps>
```

## Lead kickoff
```
You are ashura:lead for <Tn> "<title>" in run <run>.
mode: new | resume
worktree: <wt abs path>
branch: <branch>
base_ref: <working or parent branch>
task_dir: <task_dir abs path>
state: <run_dir>/state.md            (read Environment + House rules)
subtask: <run_dir>/subtasks.md#<Tn>  (goal, done evidence, file boundary)
qa_log: <task_dir>/qa.md
templates: <this file's abs path>
Return only your STATUS block (first line STATUS: <TOKEN>, ≤15 lines; QUESTIONS: full round).
```

## Status board
```
| T | title | phase | status | waiting on |
|---|---|---|---|---|
```
Phase ∈ plan, red, green, verify, review, verified, handed off, merged, paused, failed.

## PR body (written by verifier to `<task_dir>/pr-body.md`, for the user to paste)
```markdown
## Why
<1–3 lines: problem and goal>

## What changed
- <bullet per meaningful change>

## Scope
Subtask <Tn> of Ashura run <run>. Boundary: <globs>. Out of scope: <items>.

## Tradeoffs
<what was chosen over what, and why>

## Blast radius
<callers / modules / users affected; risk level>

## Verification
- <done evidence command> → <result>
- <test command> → <N passed>
- AC1 → <test id / evidence>
- Manual: <manual check, if any>
```
≤40 lines. End with the PR attribution line from House rules.
