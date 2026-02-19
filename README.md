[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

# OpenClaw Cost Guardian

An advisory skill for [OpenClaw](https://github.com/openclaw/openclaw) that recommends cost-saving defaults for cron schedules, model selection, and polling patterns. **This skill is recommendation-only — it never executes commands or modifies files.**

## The Problem

OpenClaw's antfarm workflows default to 5-minute polling intervals using your primary (expensive) model. A single workflow with 5 agents burns ~1,440 API calls/day — even when idle. That's $10+/day on heartbeats alone.

## What This Skill Does

- **Flags expensive polling intervals** — recommends 2-4 hours for daily workflows, flags anything under 5 minutes
- **Recommends cheaper heartbeat models** — suggests using cheapest available model for peek/NO_WORK checks
- **Shows cost estimates** — calculates daily/monthly cost before you create any cron job
- **Budget alerts** — warns at $5/day, critical at $15/day
- **Recommends disabling completed workflow pollers** — suggests stopping polling after workflow finishes

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

The skill activates automatically when you or another skill tries to create cron jobs or workflows and provides recommendations. You can also ask OpenClaw directly:

- "Audit my costs"
- "How much am I spending?"
- "Review my cron jobs for waste"

## Managing Cron Jobs

Refer to your OpenClaw gateway documentation for commands to list, disable, and configure cron jobs.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get involved.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

MIT
