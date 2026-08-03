# Loop contract

> A loop is not "a task, repeated". It is a contract with six clauses. Fill one of
> these in *before* you start a loop. If any clause is blank, you are not running a
> loop — you are running an open-ended agent with a billing surprise waiting at the
> end of the month.
>
> Six-clause shape after Developers Digest's framing of Boris Cherny's agent routines.

```text
TRIGGER  → when does a tick fire?         e.g. every 15m | on PR comment | on CI failure
SCOPE    → what may it touch?             e.g. open PRs I authored, repo X only
ACTION   → what does one tick do?         e.g. run tests, fix lint, respond to review
BUDGET   → the ceiling per tick           e.g. max 3 sub-agents, 50k tokens
STOP     → the halt conditions            e.g. all PRs green | 10 iterations | $5 spent
REPORT   → where the result goes          e.g. append summary to Slack #eng-bots
```

## This loop

<!-- Copy the block above and fill it in for the specific loop you are about to run.
     scripts/loop.sh reads the matching environment variables; keep them in sync. -->

| Clause      | Value                                                       | Env var in `scripts/loop.sh` |
| ----------- | ----------------------------------------------------------- | ---------------------------- |
| **TRIGGER** | <!-- e.g. every 15 minutes -->                              | `LOOP_INTERVAL_SECONDS`      |
| **SCOPE**   | <!-- e.g. items in tasks/todo.md, this worktree only -->    | _(declared in `VISION.md`)_  |
| **ACTION**  | <!-- e.g. implement next todo item, verify, commit -->      | _(declared in `PROMPT.md`)_  |
| **BUDGET**  | <!-- e.g. max 12 iterations, ~$5, 90 minutes wall clock --> | `MAX_ITER`, `MAX_USD`, `MAX_WALL_SECONDS` |
| **STOP**    | <!-- e.g. ALL DONE, BLOCKED, no-progress×3, budget hit -->  | `NO_PROGRESS_LIMIT` (+ above) |
| **REPORT**  | <!-- e.g. state/journal.log + final state/summary.md -->    | `REPORT_PATH`                |

## Why each clause exists

- **TRIGGER and SCOPE** keep the loop from doing the wrong work, or work you never
  asked for. Scope is mostly declared in `VISION.md`; the trigger is the cadence.
- **ACTION** is the single unit of work per tick. Narrow it. A wide action is how a
  loop quietly rewrites half the codebase overnight.
- **BUDGET and STOP** are the difference between a background process and a runaway.
  The costly part of agentic coding in 2026 is no longer writing code — it is
  managing the loop. Cap iterations, detect no-progress, and set a dollar ceiling.
- **REPORT** is how you trust the thing you did not watch. No report, no trust.
