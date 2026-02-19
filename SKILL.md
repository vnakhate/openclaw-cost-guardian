---
name: cost-guardian
description: "Prevents wasteful token spend. Automatically reviews cron jobs, polling intervals, and model selection before they go live. Enforces cost budgets and flags expensive patterns. Invoke when creating cron jobs, workflows, or when asked about costs/usage."
user-invocable: true
---

# Cost Guardian

Prevents runaway API costs by enforcing smart defaults on cron schedules, model selection, and polling patterns.

## Rules — ALWAYS enforce these

### 1. Cron Polling Intervals

Before creating or modifying ANY cron job, apply these minimum intervals:

| Task Type | Minimum Interval | Rationale |
|-----------|-----------------|-----------|
| Daily workflow (job hunt, reports, digests) | **4 hours** | Work is created once/day, no need to poll constantly |
| Monitoring/alerting | **15 minutes** | Reasonable for non-critical monitoring |
| Real-time chat/response | **No cron needed** | Use event-driven, not polling |
| Heartbeat/health check | **1 hour** | Just checking if things are alive |
| Antfarm workflow agents | **2 hours** | Steps complete in minutes, polling can be lazy |

**NEVER allow polling intervals under 5 minutes** unless the user explicitly says "I understand this will cost $X/day and I want it anyway."

### 2. Model Selection for Cron Jobs

Cron jobs that just check for work (peek/heartbeat) should use the **cheapest available model**, not the primary.

| Task | Recommended Model |
|------|------------------|
| Heartbeat / peek for work | Cheapest fallback (e.g., `openrouter/z-ai/glm-5` or free flash model) |
| Actual work execution | Primary model is fine |
| Daily scheduled tasks | Primary model is fine |

**NEVER use Opus or Sonnet for heartbeat polling.** If a cron payload contains "peek", "NO_WORK", or "HEARTBEAT_OK", it MUST use a cheap model.

### 3. Cost Estimation — Show Before Creating

Before creating any cron job, calculate and display estimated daily/monthly cost:

```
Estimated cost:
  Calls/day: {24 * 60 / interval_minutes}
  ~tokens/call: {estimate based on payload size}
  Model rate: {input + output per M tokens}
  Daily cost: ${calls * tokens * rate}
  Monthly cost: ${daily * 30}
```

Ask for user confirmation if monthly estimate exceeds $5.

### 4. Budget Alerts

Track cumulative spend via OpenRouter API:
```bash
curl -s https://openrouter.ai/api/v1/auth/key \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" | python3 -c "
import json,sys
d=json.load(sys.stdin)['data']
print(f'Today: ${d[\"usage_daily\"]:.2f}')
print(f'This week: ${d[\"usage_weekly\"]:.2f}')
print(f'This month: ${d[\"usage_monthly\"]:.2f}')
"
```

**Alert thresholds:**
- Daily spend > $5 → WARN
- Daily spend > $15 → CRITICAL — suggest stopping non-essential crons
- Weekly spend > $30 → Review all cron jobs and suggest optimizations

### 5. Antfarm Workflow Guard

When installing antfarm workflows:
- Override default polling from 5min to **2 hours** (`7200000` ms)
- Use cheap model for peek/heartbeat payloads
- Only use primary model for actual step execution (claim + work)
- After workflow completes, **disable polling crons** for that workflow

### 6. Audit Command

When user asks to audit costs, run:

1. Check OpenRouter spend (API call above)
2. List all cron jobs: `openclaw cron list`
3. Flag any job polling more frequently than its category minimum
4. Flag any heartbeat job using an expensive model
5. Calculate projected monthly cost
6. Recommend specific changes

## Example Intervention

**User says:** "Set up a cron to check for new emails every minute"

**Cost Guardian response:**
> Checking every minute with Sonnet 4.6 would cost ~$8-15/day (~$250-450/month).
>
> Recommended: Every 15 minutes with GLM-4.7-Flash (free).
> That's $0/day and you'll still catch emails within 15 min.
>
> Want me to set it up at 15min with the free model?

## Quick Reference

```bash
# Check current spend
curl -s https://openrouter.ai/api/v1/auth/key -H "Authorization: Bearer $OPENROUTER_API_KEY"

# List cron jobs
openclaw cron list

# Disable a job
openclaw cron disable <job-id>

# Update interval (via jobs.json when gateway stopped)
# Edit ~/.openclaw/cron/jobs.json → everyMs field
```
