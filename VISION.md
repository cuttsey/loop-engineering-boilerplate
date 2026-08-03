# Vision

> The north star. Every loop iteration re-reads this file so that intent is never
> re-derived from a growing conversation. Keep it short, durable, and honest about
> what "done" means. If a tick cannot tell whether its work serves this file, the
> file is too vague.

## What we are building

<!-- One paragraph. The product or outcome, in plain terms. Replace this. -->

_Example: A scheduling service that ingests calendar webhooks, resolves conflicts
against a team availability model, and proposes meeting slots via an HTTP API._

## Why it matters

<!-- The problem this removes. Who is worse off if it does not exist. -->

## Constraints that do not move

<!-- The boundaries the agent must respect on every tick. Be specific. -->

- **Stack**: <!-- e.g. TypeScript, Node 20, PostgreSQL -->
- **Public surface**: <!-- e.g. only the documented HTTP routes; no breaking changes -->
- **Dependencies**: <!-- e.g. no new runtime dependencies without an entry in lessons.md -->
- **Security**: <!-- e.g. no secrets in source; no outbound calls to undeclared domains -->
- **Performance**: <!-- e.g. p95 < 200ms on the conflict resolver -->

## Definition of done

<!--
The verifiable conditions under which the loop should STOP. These must be
machine-checkable wherever possible, because they are what the loop reads to
decide it is finished. "Looks good" is not a definition of done.
-->

- [ ] All items in `tasks/todo.md` are checked or explicitly marked `BLOCKED`.
- [ ] `npm test` exits zero. <!-- replace with your gate -->
- [ ] Type check exits zero.
- [ ] No `TODO(loop)` markers remain in shipped code.

## Explicitly out of scope

<!-- Name the things you do NOT want a self-prompting agent to wander into. -->

- <!-- e.g. database schema migrations (human-reviewed only) -->
- <!-- e.g. changes to billing logic -->
