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
   with `depends_on`. Two writers never share a file.

## Ownership
4. **One owner per branch.** One lead owns one worktree and one branch. Inside a worktree only one
   writing agent runs at a time; read-only agents may overlap.
5. **Verifier is not the author.** Reviewer and verifier are spawned fresh and stateless. They read
   the diff, not anyone's summary of it. Green tests are input to the verdict, not the verdict.

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
   Behaviour no automated test can reach (UI look, a click path) is a `manual` row: the verifier
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
16. **Handing off.** Delivery is a verified local branch plus `pr-body.md`. Conventional commit
    titles (`feat(scope): …`). Body sections: Why / What changed / Scope / Tradeoffs / Blast radius
    / Verification, ≤40 lines. The user merges into the working branch, never into `main` straight
    from a subtask. Ashura never pushes and never calls GitHub.
