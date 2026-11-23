# Ashura principles

Rules for Ashura and every agent it spawns. Same as Sonny Boy's (adapted from pstack by Lauren
Tan), minus the PR and browser parts.

## Scoping
1. **Restate before splitting.** Before decomposing, state the goal in your own words and the
   smallest change that meets it. Decompose that, not the wish list.
2. **No evidence, not scoped.** Every subtask names its **done evidence** (a command and the output
   that proves it is done) and its **file boundary** (globs it may touch). A subtask without both
   goes back to the decomposer.
3. **Separate before serializing.** Overlapping file boundaries → merge the subtasks, or chain them
   with `depends_on`. Two writers never share a file. A chained child starts after its parent
   lands on the run branch; branches are never stacked.

## Ownership
4. **One owner per branch.** One lead owns one worktree and one branch. Inside a worktree only one
   writing agent runs at a time; read-only agents may overlap.
5. **Tester, coder and checker are three agents.** A fresh tester writes the red tests from the
   plan alone; the coder never edits them. The reviewer is spawned fresh and stateless, once per subtask
   (plus bounded re-checks of fix diffs). It reads the diff, not anyone's summary of it. Green tests
   are input to the verdict, not the verdict. One independent check, not three.

## Asking
6. **Never block on a fact.** Look facts up (code, docs, git, running commands). Ask the user only
   for product decisions, irreversible steps, and dead ends after a replan.
7. **Never answer for the user.** Relay questions with options and a recommendation. Pass "Other"
   text verbatim.

## Proof
8. **A finding survives only when a run proves it.** Reviewer BLOCKER/MAJOR needs `proof:` — a
   command plus its output, or a `path:line` trace confirmed by reading. Unproven → `[UNVERIFIED]`,
   never blocking.
9. **Prove it works; test behaviour.** Tests first: failing tests from the approved design are
   committed before the code that makes them pass (`test(...)` commit, then `feat/fix(...)`).
   Behaviour no automated test can reach (UI look, a click path) is a `manual` row: the reviewer
   lists it and the user checks it by hand at the handoff gate.
10. **Attack the premise.** After 2 failed fixes on one step, the advisor tests the shared
    assumption behind them instead of proposing fix #3.
11. **Retry by cause, max 2, then replan.** Classify the failure (env, flaky, wrong plan, wrong
    test, code bug) and retry only what the cause allows. Third failure → escalate.

## Record
12. **Show your work.** Append one row to `decisions.tsv` (run level and task level) for every
    decision: `ts, phase, decision, why, evidence, result`. Append only.
13. **Pause safely.** Pausing = finish the current step, `wip:` commit on the subtask branch, write
    `status.md` (done, next, open questions). A fresh session resumes from files alone.

## Output
14. **Guard the context window.** Kickoffs pass file paths, never file contents. Agents return
    only their STATUS block; detail lives in task files. Ashura reads task files only when a gate
    needs them.
15. **Status only on tracked change.** Post the status board when a tracked field changes, not on
    every notification. The close report states: landed, merged, dropped, waiting.
16. **Handing off.** Delivery is one verified local run branch plus `pr-body.md`. Conventional commit
    titles (`feat(scope): …`). Body sections: Why / What changed / Scope / Tradeoffs / Blast radius
    / Verification, ≤40 lines. Ashura merges checked subtask branches only into the local run branch
    `ashura/<run>`. The user merges the run branch into the working branch, never into `main`
    straight. Ashura never pushes and never calls GitHub.

## Speed
17. **Run each test once per commit.** Agents run targeted tests only. The full suite runs once per
    batch, on the run branch, by Ashura. Every run is cached in `test-runs.tsv` by commit SHA +
    scope; the same SHA and scope is never run twice.
18. **Hand over, do not re-research.** The planner writes `research.md` → `## Brief`. Later agents
    read it and go straight to the files it names.
19. **Bound the loops.** Max 2 fix rounds per subtask; a re-check covers only the fix diff. Then the
    user decides. MINOR findings never start a round.
20. **Write only what someone reads.** No per-step status files; `status.md` only on pause;
    `decisions.tsv` only for decisions; one `pr-body.md` per run.
