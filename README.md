# OpenClaw Cost Guardian

A skill for [OpenClaw](https://github.com/openclaw/openclaw) that prevents runaway API costs by enforcing smart defaults on cron schedules, model selection, and polling patterns.

## The Problem

OpenClaw's antfarm workflows default to 5-minute polling intervals using your primary (expensive) model. A single workflow with 5 agents burns ~1,440 API calls/day — even when idle. That's $10+/day on heartbeats alone.

## What This Skill Does

- **Enforces minimum polling intervals** — blocks cron jobs under 5 minutes, recommends 2-4 hours for daily workflows
- **Downgrades heartbeat models** — peek/NO_WORK checks use cheapest available model, not Sonnet/Opus
- **Shows cost estimates** — calculates daily/monthly cost before creating any cron job
- **Budget alerts** — warns at $5/day, critical at $15/day
- **Auto-disables completed workflow pollers** — stops polling after workflow finishes

## Install

```bash
mkdir -p ~/.openclaw/skills/cost-guardian
curl -o ~/.openclaw/skills/cost-guardian/SKILL.md \
  https://raw.githubusercontent.com/vnakhate/openclaw-cost-guardian/main/SKILL.md
```

Then restart your gateway:

```bash
openclaw gateway restart
```

## Cost Impact

| Metric | Without Skill | With Skill |
|--------|--------------|------------|
| Heartbeats/day (5-agent workflow) | ~1,440 | ~30 |
| Daily idle cost | ~$10 | ~$0.20 |
| Monthly idle cost | ~$300 | ~$6 |

## Usage

The skill activates automatically when you or another skill tries to create cron jobs or workflows. You can also ask OpenClaw directly:

- "Audit my costs"
- "How much am I spending?"
- "Review my cron jobs for waste"

## License

MIT
