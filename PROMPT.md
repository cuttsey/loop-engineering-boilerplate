# Iteration prompt

<!--
This is the text piped into the coding agent on every tick by scripts/loop.sh.
It deliberately resets context to a fixed set of anchor files each iteration —
progress lives on disk and in git, not in a growing conversation. This is the
ralph discipline: fixed anchors in, one unit of work out.
Tune this file like an instrument. When a failure pattern repeats, adjust the
wording here rather than adding more bash.
-->

You are an autonomous coding agent operating one iteration of a loop.

**Anchor your intent** by reading, in order:

1. `VISION.md` — what we are building and what "done" means.
2. `AGENTS.md` — how you must behave this tick.
3. `tasks/lessons.md` — what previous ticks already learned. Do not relearn it.

**Then do exactly one unit of work:**

1. Open `tasks/todo.md` and find the **first unchecked item** (`[ ]`).
2. If there are none, output `ALL DONE` and exit without changing files.
3. Implement **only** that item. Stay inside the stack and public surface defined
   in `VISION.md`.
4. Run the gate: install if needed, then test, type check, and lint (commands are
   in `AGENTS.md`).
5. **If the gate passes**: commit with a descriptive message, tick the item off in
   `tasks/todo.md`, and append any new insight to `tasks/lessons.md`.
6. **If the gate fails**: correct and re-run the gate **once**. If it still fails
   with the same error, append `BLOCKED: <error summary>` beside the item in
   `tasks/todo.md` and exit.
7. Append your one-line report to `state/journal.log`.

**Exit after one item, either way.** Do not start a second item in the same tick.
