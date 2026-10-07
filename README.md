# Kingmaker

A Hermes Agent skill: keep your main model on the strongest model inside a per-task
budget you choose — using the official [Hermes Index](https://portal.nousresearch.com/bench)
(measured average `$/task`, Hermes Agent harness) instead of vibes.

One-time setup asks three questions. After that a **no_agent cron job** runs a plain
script: no model runs on a tick, so checking costs nothing, and a healthy tick is silent.

## Install

```
hermes skills install Salt-555/kingmaker/skills/kingmaker
```

## Requirements

- Hermes Agent with cron jobs (`cronjob_manage`)
- the `nous` provider configured — the Hermes Index ranks nous-routed models
- `python3` and `curl` on `PATH`, Linux or macOS

## What setup asks

1. **Budget** — the live board is sampled first, so you see what each price bracket
   actually buys today (under $0.10/task vs under $2 reads very differently).
2. **Apply or notify** — switch your main model automatically when a better in-budget
   option appears, or just tell you the change and the exact command.
3. **Cadence** — daily, twice daily, weekly, monthly, or any cron expression.

Then it installs the script and creates the job. That is the whole user surface.

## What a tick does

Fetches the Hermes Index leaderboard, ranks it, and picks the highest-scoring
nous-routed model whose measured average `$/task` is **strictly under** your budget.

- Provisional rows (estimates, marked `*` on the page) never qualify.
- A row whose provider identity cannot be mapped exactly **blocks** the switch and is
  surfaced, rather than being guessed at.
- In apply mode the candidate is smoke-tested (chat, tool calling, structured output),
  applied with `hermes config set model.default`, then verified against the effective
  config and the next job's scheduler resolution — rolled back if either disagrees.
- Rate limits are transient: retried with backoff, then deferred.

Empty output means there was nothing to report. The only Hermes setting this skill
ever writes is the main model.

## Layout

```
skills/kingmaker/
├── SKILL.md                    the setup instructions (the wizard)
└── references/
    ├── kingmaker-job.py        the script the cron job runs
    └── cron-job.md             the no_agent job template
```

## License

MIT — see [LICENSE](LICENSE).