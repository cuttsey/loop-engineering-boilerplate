# Agent operating rules

> Read on every iteration alongside `VISION.md`. `VISION.md` says *where* to go;
> this file says *how* to behave while getting there. Claude Code reads `CLAUDE.md`;
> Codex and several other harnesses read `AGENTS.md`. This repository keeps the rules
> here and lets `CLAUDE.md` point at this file, so there is one source of truth.

## Plan before you act

- Default to **plan mode**. Before changing files, write the plan as the next
  unchecked block in `tasks/todo.md`, then execute against it.
- Do **one discrete unit of work** per iteration. Implement it, verify it, exit.
  Do not batch unrelated changes into a single tick.

## The standard

- Hold every change to a **senior engineer's standard**: small, reviewable diffs;
  named functions over cleverness; no dead code; no commented-out blocks left behind.
- If you introduce a pattern, a dependency, or a non-obvious decision, record it in
  `tasks/lessons.md` so the next tick does not relearn it.

## Verification is mandatory

A tick that writes code without running the gate is an open loop, which is a machine
for producing confident mistakes. Every tick must:

1. **Write** the change.
2. **Run** the gate (`npm test`, type check, lint — see commands below).
3. **Read** the result.
4. **Correct** if it failed, or commit if it passed.

## Commands

<!-- Replace with your real commands. The loop runner and the agent both rely on these. -->

| Purpose      | Command                |
| ------------ | ---------------------- |
| Install      | `npm ci`               |
| Test         | `npm test`             |
| Type check   | `npm run typecheck`    |
| Lint         | `npm run lint`         |
| Build        | `npm run build`        |

## Delegation

- For work that fans out (e.g. the same fix across many files or PRs), dispatch
  **sub-agents in isolated worktrees** rather than doing it serially in one context.
- A sub-agent gets a single, closed task and reports back. The parent tick decides
  what to do with the result; it does not merge blindly.

## Guardrails the agent must respect

- Never edit `VISION.md` or files under `state/` — those are owned by the human and
  the loop runner respectively.
- Never run destructive git operations (`push --force`, `reset --hard` on shared
  branches, history rewrites).
- If the same error recurs **twice** with no progress, stop and write `BLOCKED`
  with the error into `tasks/todo.md`. Do not keep agreeing with yourself.
- Stay inside the declared stack and public surface in `VISION.md`.

## Reporting

At the end of each tick, append a one-line entry to `state/journal.log`:
`<iso-timestamp> | iter=<n> | <committed|blocked|noop> | <short summary>`.
