# OpenClaw Heartbeat

> Last updated: 2026-03-20

## What It Is

A background daemon that runs periodic agent turns without you prompting it. This is what makes OpenClaw autonomous rather than reactive.

Think of it as cron for your AI agent.

## Key Settings

| Setting | Default | Notes |
|---|---|---|
| `agents.defaults.heartbeat.every` | 30 minutes | Set to `0m` to disable |
| `heartbeat.activeHours` | Not set | Prevents overnight runs |
| Heartbeat target | `"last"` | Where responses go: `"none"`, `"last"`, or specific channel |

## HEARTBEAT.md

A plain-English checklist of routine tasks the agent should perform on each heartbeat cycle.

If nothing needs attention, the agent replies `HEARTBEAT_OK`.

## Example Use Cases

- Check email and summarise anything urgent
- Monitor a Slack channel and flag action items
- Review calendar and prep for upcoming meetings
- Check project status and update dashboards
- Run routine data pulls or reports

## TODO: Research Gaps

- [ ] HEARTBEAT.md syntax and best practices
- [ ] Active hours configuration examples
- [ ] Heartbeat cost implications (API token usage)
- [ ] Heartbeat + multi-agent patterns
- [ ] Error handling when heartbeat tasks fail
- [ ] Monitoring heartbeat health
