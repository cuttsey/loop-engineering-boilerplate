# Default loop prompt (`/loop`)

<!--
Claude Code uses a file named loop.md in the project to replace the built-in
maintenance prompt for a bare `/loop` invocation. This is the standing-maintenance
prompt, distinct from PROMPT.md (which drives the ralph-style task loop in
scripts/loop.sh). Keep this one focused on continuous upkeep, not feature work.
-->

Maintain this repository continuously. On each tick:

1. Read `VISION.md` and `AGENTS.md` so you act in scope.
2. Check open pull requests authored by the repository owner in this repo.
   - If CI is failing, reproduce locally and fix it in a worktree, then push.
   - If there are unresolved review comments, address them in a worktree and push.
3. Check the latest deploy or build status; if it is broken, diagnose and open a fix.
4. If nothing needs attention, do nothing and wait for the next tick.

Respect every guardrail in `AGENTS.md`. Never force-push or rewrite shared history.
Stop and surface a summary if you would otherwise act outside `VISION.md`.
