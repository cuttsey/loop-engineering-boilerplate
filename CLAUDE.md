# Claude Code rules

This project keeps a single source of operating rules in **[`AGENTS.md`](./AGENTS.md)**
so that Claude Code, Codex, and any other harness behave identically.

**Read [`AGENTS.md`](./AGENTS.md) and [`VISION.md`](./VISION.md) at the start of every tick.**

Quick reference:

- Plan in `tasks/todo.md` before acting; one unit of work per iteration.
- Verify with the gate (see commands in `AGENTS.md`) before committing.
- Record decisions and dead ends in `tasks/lessons.md`.
- Stop and write `BLOCKED` after two failures with no progress.
