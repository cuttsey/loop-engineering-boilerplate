# Contributing

Contributions are welcome. This is a small, deliberately minimal scaffold, so the
bar for additions is that they keep it minimal.

## Ground rules

- **Keep it portable.** The scaffold should run with nothing beyond `bash` and `git`.
  Anything that needs a heavier toolchain belongs in an example, not the core.
- **Keep the guardrails honest.** Changes to `scripts/lib/guardrails.sh` must come
  with a corresponding check. The point of those functions is that they can say
  *no*; a change that weakens that is unlikely to be merged.
- **Document intent, not mechanics.** The anchor files explain *why* each part
  exists. New files should do the same.

## Before opening a pull request

Run the syntax checks:

```bash
bash -n scripts/loop.sh
bash -n scripts/lib/guardrails.sh
```

If you change a guardrail, include a short shell snippet in the pull request that
demonstrates the new behaviour passing and failing.

## Reporting issues

Open an issue describing the loop you were running, the configuration
(`MAX_ITER`, `MAX_USD`, and so on), and what the loop did versus what you expected.
Redact anything that identifies a private repository, a customer, or a credential.
