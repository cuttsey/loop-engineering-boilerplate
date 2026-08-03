# Loop engineering boilerplate

A minimal, opinionated scaffold for **loop engineering**: writing the program that
prompts a coding agent on a schedule, rather than typing prompts into one by hand.
It assembles the pieces the discipline rests on — anchor files, a loop contract,
verification, and the three hard stops — into a repository you can clone, fill in,
and run.

Everything here depends on nothing beyond `bash` and `git`. Use it as a starting
point, delete what you do not need, and replace the placeholders with your own
project's detail.

The structure follows the synthesis in [Loop engineering: how to design coding
agent loops that run while you sleep](https://explainx.ai/blog/loop-engineering-coding-agents-claude-code-guide-2026)
(Yash Thakker, ExplainX, 9 June 2026), which in turn draws on the practitioners who
named the shift. The intent here is to make that synthesis *buildable*.

## The idea in one paragraph

The job moves up one level of abstraction. The operator stops sitting inside the
loop typing prompts and becomes the author of the loop; the model becomes a
subroutine the loop calls. This framing comes from
[Boris Cherny](https://workos.com/blog/acquired-unplugged-boris-cherny), creator of
Claude Code, who described running loops that prompt the agent and decide what to
do, and from [Peter Steinberger's June 2026 note](https://steipete.me/posts/just-talk-to-it)
that one should be designing the loops that prompt agents. The oldest form is the
academic reason–act cycle of the [ReAct paper](https://arxiv.org/abs/2210.03629)
(2022); the practical baseline is the [ralph loop](https://ghuntley.com/ralph/)
(Geoffrey Huntley, 2025), whose whole innovation was discipline rather than
cleverness — fixed anchor files in, one unit of work out, progress kept on disk and
in git rather than in a growing conversation.

## What is in the box

| Path                        | Role                                                                        |
| --------------------------- | --------------------------------------------------------------------------- |
| `VISION.md`                 | North star. What is being built, the constraints, and what "done" means.    |
| `AGENTS.md`                 | Operating rules read every tick. The single source of behaviour.            |
| `CLAUDE.md`                 | Thin pointer to `AGENTS.md` so Claude Code and other harnesses agree.       |
| `PROMPT.md`                 | The prompt piped into the agent on each iteration of the ralph loop.        |
| `loop.md`                   | Custom default prompt for Claude Code's bare `/loop` maintenance command.   |
| `loop-contract.md`          | The six-clause contract: trigger, scope, action, budget, stop, report.      |
| `tasks/todo.md`             | The work queue. One small, verifiable item per tick.                        |
| `tasks/lessons.md`          | The self-improvement file. What each tick learned, so the next is cheaper.  |
| `scripts/loop.sh`           | A ralph loop with the three hard stops bolted on.                           |
| `scripts/lib/guardrails.sh` | The functions that say "no": ceilings, no-progress detection, terminal states. |
| `state/`                    | Runtime artefacts: the per-tick journal, the final summary, the cost ledger (all git-ignored). |

## The two halves of a loop

A useful loop has a designed half and a half that can push back. Designing the loop
is only the first half; the second is putting something inside it that can say *no* —
a test, a type check, a real error. A loop with nothing to push back is the agent
agreeing with itself on repeat. In this scaffold:

- **The designed half** lives in `VISION.md`, `AGENTS.md`, `PROMPT.md` and the loop
  contract — intent, behaviour, and the unit of work.
- **The half that says no** lives in the test and type-check gate (declared in
  `AGENTS.md`) and in `scripts/lib/guardrails.sh`, which enforces the stops.

This is the open-versus-closed distinction the source article draws: an open loop
writes until it declares itself done and is fit only for demos; a closed loop runs
the gate after each write and is what ships.

## The loop contract

Fill in [`loop-contract.md`](./loop-contract.md) before running anything. A loop is
not "a task, repeated" — it is six clauses:

```text
TRIGGER  -> when a tick fires
SCOPE    -> what it may touch
ACTION   -> what one tick does
BUDGET   -> the ceiling per tick / per run
STOP     -> the halt conditions
REPORT   -> where the result goes
```

The six-clause shape is from
[Developers Digest's account](https://www.developersdigest.tech/blog/codex-loops-boris-cherny-agent-routines)
of Cherny's agent routines.

## The three hard stops

Once the model writes code for almost nothing, cost moves to *managing the loop*.
Every serious write-up converges on three ceilings, all implemented in
`scripts/lib/guardrails.sh` and wired into `scripts/loop.sh`:

1. **Maximum iteration count** — `MAX_ITER`. A bare ralph loop has no ceiling unless
   one is added.
2. **No-progress detection** — `NO_PROGRESS_LIMIT`. Stop when the working tree and
   the todo file stop changing for N ticks in a row, so the agent cannot quietly
   agree with itself.
3. **Budget ceiling** — `MAX_USD` plus a `MAX_WALL_SECONDS` backstop. True token
   accounting must come from the harness or API; a command-line tool does not expose
   it reliably, so the dollar ceiling reads an optional cost ledger and the
   wall-clock ceiling is the dependable fallback.

The cautionary receipt the source article cites: one large engineering organisation
capped per-engineer monthly spend on agentic coding tools after exhausting its
annual AI budget in four months. Caps are not optional.

## Quick start

### Option A — Claude Code, one slash command

The fastest on-ramp needs no script at all. With `loop.md` in place to define the
maintenance behaviour, inside Claude Code:

```text
/loop babysit all my PRs. Auto-fix build issues, and when comments come in, use a worktree agent to fix them.
```

`/loop` accepts a cadence (`/loop 5m /babysit`) or, with the interval omitted, lets
the agent choose a delay between one minute and one hour based on what it observed.
Press **Esc** while it waits to stop it. See the
[Claude Code scheduled-tasks documentation](https://code.claude.com/docs/en/scheduled-tasks).

### Option B — the ralph loop with guardrails

1. Fill in `VISION.md`, the commands in `AGENTS.md`, and the queue in `tasks/todo.md`.
2. Complete `loop-contract.md` and set the matching values at the top of
   `scripts/loop.sh` (or pass them as environment variables).
3. Run it **inside an isolated git worktree or a container** — the runner invokes the
   agent with permission prompts disabled so it can work unattended, which is only
   acceptable in a sandbox you are willing to lose:

   ```bash
   MAX_ITER=12 MAX_USD=5 ./scripts/loop.sh
   ```

4. Watch the first two ticks. Confirm the agent reads `VISION.md` and runs the gate
   before it commits. Then leave it.

The runner stops on any of: the queue emptying (`ALL DONE`), a `BLOCKED` marker, the
iteration ceiling, the no-progress limit, the dollar budget, or the wall-clock
budget — whichever comes first — and writes a summary to `state/summary.md`.

## Design choices

A few decisions are baked in, and they are worth stating so they can be changed
deliberately rather than by accident.

- **Plan before acting.** `AGENTS.md` defaults the agent to plan mode and asks it to
  record the plan in `tasks/todo.md` before changing files. This keeps intent
  legible and reviewable rather than buried in a transcript.
- **One unit of work per tick.** Narrow actions are what stop a loop quietly
  rewriting half a codebase overnight. The prompt enforces a single item per
  iteration and an exit either way.
- **A self-improvement file.** `tasks/lessons.md` exists so that what a tick learns
  is not paid for twice. A loop that records its dead ends gets cheaper over time; a
  loop that re-derives everything does not.
- **Skills over re-described prompts.** A loop calling a library of sharp, named
  recipes compounds; a loop that re-explains the same workflow each tick burns
  tokens. As repeated workflows harden into skills, invoke them from `loop.md` or
  `PROMPT.md` rather than restating them.
- **Fan-out via sub-agents in worktrees.** Work that repeats across many files or
  pull requests is delegated to isolated sub-agents rather than done serially in one
  context, with the parent tick deciding what to do with each result.

## Verifying the scaffold

The guardrail functions ship with no external dependencies. Sanity-check the scripts
before trusting them:

```bash
bash -n scripts/loop.sh && bash -n scripts/lib/guardrails.sh
```

## Using this as a template

On GitHub, the repository can be marked as a **template repository** in its settings,
which gives others a "Use this template" button to generate their own copy with a
clean history. Either way, after cloning, the placeholders in `VISION.md`,
`AGENTS.md`, and `tasks/todo.md` are the first things to replace.

## Licence

Released under the [MIT Licence](./LICENSE). The copyright line names the original
author; change it to suit your fork.

## Credits

This scaffold is an independent implementation of ideas synthesised by others. Credit
belongs to the practitioners and writers cited below, not to this repository, which
only arranges their conclusions into runnable files.

## References

**Primary synthesis**

- [Loop engineering: how to design coding agent loops that run while you sleep](https://explainx.ai/blog/loop-engineering-coding-agents-claude-code-guide-2026) — Yash Thakker, ExplainX, 9 June 2026. The article this scaffold is built from.

**Primary sources it rests on**

- [Boris Cherny at WorkOS Acquired Unplugged](https://workos.com/blog/acquired-unplugged-boris-cherny) — 2 June 2026. The definition of a loop and the three-stage ladder.
- [Peter Steinberger — Just talk to it](https://steipete.me/posts/just-talk-to-it) — the agentic engineering workflow behind the June 2026 note.
- [Geoffrey Huntley — the Ralph Wiggum loop](https://ghuntley.com/ralph/) — July 2025. "Ralph is a bash loop"; the discipline this runner preserves.
- [HumanLayer — a brief history of ralph](https://www.humanlayer.dev/blog/brief-history-of-ralph) — context on the loop's evolution.
- [ReAct: synergizing reasoning and acting in language models](https://arxiv.org/abs/2210.03629) — Yao et al., 2022. The original reason–act cycle.
- [Developers Digest — Codex loops and Boris Cherny's agent routines](https://www.developersdigest.tech/blog/codex-loops-boris-cherny-agent-routines) — the loop-contract clauses.

**Tooling**

- [Claude Code — run prompts on a schedule (`/loop`)](https://code.claude.com/docs/en/scheduled-tasks)
- [Claude Code — bundled skills](https://code.claude.com/docs/en/skills)
- [Gas Town (Steve Yegge)](https://github.com/gastownhall/gastown) — a continuous orchestration loop coordinating many agents, for when a single ralph loop is outgrown.

---

_The view counts and thread citations in the source article are from 8–9 June 2026
and should be verified against the upstream posts before being relied upon in any
production decision._
