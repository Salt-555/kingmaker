# Kingmaker

A Hermes Agent skill: keep your main model on the best-value model inside a per-task
budget you choose — using the official [Hermes Index](https://portal.nousresearch.com/bench)
(measured average `$/task`, Hermes Agent harness) instead of vibes.

One-time setup asks your budget and what a point of intelligence is worth to you, plus
apply-or-notify, cadence, and where notices go. After that a **no_agent cron job** runs a
plain script: no model runs on a tick, so checking costs nothing, and a healthy tick is
silent.

## Install

```
hermes skills install Salt-555/kingmaker/skills/kingmaker
```

## Requirements

- Hermes Agent with cron jobs (`cronjob_manage`)
- the `nous` provider configured — the Hermes Index ranks nous-routed models
- `python3` and `curl` on `PATH`, Linux or macOS

## What setup asks

The live board is sampled first, so you see what today's prices actually buy before you
decide anything.

1. **Budget** — the hard ceiling on measured `$/task`. It never bends.
2. **Point price** — USD per Hermes Index point, i.e. what one point of intelligence is
   worth to you. Set it low and the pick favors cheap models; set it high and it favors
   smart ones; say `none` and it always takes the smartest you can afford. This is the
   knob that catches a model that's nearly as smart for half the price.
3. **Apply or notify** — switch your main model automatically when a better in-budget
   option appears, or just tell you the change and the exact command.
4. **Cadence** — daily, twice daily, weekly, monthly, or any cron expression.
5. **Where notices go** — this chat, saved locally, or a specific channel.

Then it installs the script and creates the job. That is the whole user surface.

## What a tick does

Fetches the Hermes Index leaderboard, ranks it, and picks the highest-utility nous-routed
model whose measured average `$/task` is **strictly under** your budget, where

```
utility = index - ($/task) / (point price)
```

so the pick always lands on the board's Pareto frontier. `--usd-per-point none` restores
plain max index.

- Provisional rows (estimates, marked `*` on the page) never qualify.
- A row whose provider identity cannot be mapped exactly **blocks** the switch and is
  surfaced, rather than being guessed at.
- In apply mode the candidate is smoke-tested (chat, tool calling, structured output),
  applied with `hermes config set model.default`, then verified against the effective
  config — rolled back if it disagrees.
- Rate limits are transient: retried with backoff, then deferred.

Empty output means there was nothing to report. The only Hermes setting this skill
ever writes is the main model.

## Layout

```
skills/kingmaker/
├── SKILL.md                    the setup instructions (the wizard)
└── references/
    ├── kingmaker-job.py        the script the cron job runs
    ├── cron-job.md             the no_agent job template
    └── selection-policy.md     the selection math and the traps
```

## License

MIT — see [LICENSE](LICENSE).
