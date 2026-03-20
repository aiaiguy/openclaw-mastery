# OpenClaw Heartbeat — Comprehensive Reference

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai/gateway/heartbeat.md, docs.openclaw.ai/automation/cron-vs-heartbeat.md

## What It Is

A periodic automation mechanism built into the OpenClaw gateway. Triggers agent turns on a scheduled cadence within the agent's main session. The daemon fires, the agent checks its checklist, and either surfaces something that needs attention or responds with `HEARTBEAT_OK`.

Think of it as cron for your AI agent. No external infrastructure required.

## How a Heartbeat Cycle Works

1. Daemon fires at configured interval (default: 30 min, or 60 min for Anthropic OAuth/setup-token auth)
2. Agent receives its heartbeat prompt (default or custom)
3. If `HEARTBEAT.md` exists, it's read as a task checklist
4. Agent evaluates whether anything needs attention
5. If nothing: responds `HEARTBEAT_OK`. System strips the token and drops the reply if remaining content is 300 chars or fewer (`ackMaxChars`)
6. If something: omits `HEARTBEAT_OK` and delivers alert to configured target channel

**Key details:**
- Runs inside the gateway process. No external cron needed.
- Executes within agent's main session context unless `isolatedSession: true` is set.
- Active hours are timezone-aware. Outside the window, heartbeats defer to next tick inside the window.
- Heartbeat deliveries do not reset session idle timers.
- If `HEARTBEAT.md` contains only whitespace/headers, the run is skipped entirely (zero API cost).

## HEARTBEAT.md

Plain Markdown checklist in the agent's workspace directory. The docs call it: "your heartbeat checklist: small, stable, and safe to include every 30 minutes."

**Rules:**
- Standard Markdown format
- Short task items, reminders, or conditions
- Can include conditional logic in natural language
- Agent can update the file during normal conversations if instructed
- Keep content minimal to avoid prompt bloat
- Never include secrets (API keys, tokens, phone numbers)
- Whitespace/header-only files cause the run to be skipped (zero cost)

**Example:**
```markdown
# Heartbeat Checklist

- Check inbox for urgent emails flagged in the last 30 minutes
- Review calendar for meetings starting in the next hour
- Check Slack #alerts channel for any unresolved incidents
- If a task is overdue in the project tracker, notify me
- Only check Gmail if it's after 9am and a task is pending
- Skip calendar check on weekends
```

The default prompt instructs the agent: "Read HEARTBEAT.md if it exists. Follow it strictly. Do not infer or repeat old tasks from prior chats."

## Full Configuration Reference

All settings in `openclaw.json`. Global defaults at `agents.defaults.heartbeat`, per-agent overrides at `agents.list[].heartbeat`.

```jsonc
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",                    // Interval. "0m" disables. Default 30m (1h for OAuth)
        "model": "provider/model",         // Optional model override (use cheaper model here)
        "target": "last|none|<channel>",   // Where alerts go. "none" = internal only
        "prompt": "custom body",           // Replaces default prompt verbatim
        "lightContext": true,              // Bootstrap with HEARTBEAT.md only
        "isolatedSession": true,           // Fresh session each run (no history)
        "includeReasoning": true,          // Deliver separate "Reasoning:" messages
        "ackMaxChars": 300,                // Max chars after HEARTBEAT_OK before drop
        "directPolicy": "allow|block",     // DM suppression
        "suppressToolErrorWarnings": true,
        "activeHours": {
          "start": "09:00",               // HH:MM format
          "end": "22:00",                  // HH:MM format
          "timezone": "Australia/Melbourne" // IANA timezone
        }
      }
    }
  }
}
```

**Timezone resolution order:**
1. Explicit `activeHours.timezone`
2. `agents.defaults.userTimezone`
3. Host system timezone

**Channel-level visibility controls:**
```jsonc
{
  "channels": {
    "defaults": {
      "heartbeat": {
        "showOk": false,       // Hide HEARTBEAT_OK acknowledgements (default)
        "showAlerts": true,    // Show non-OK replies (default)
        "useIndicator": true   // Emit UI status events (default)
      }
    }
  }
}
```

If all three visibility flags are false, the heartbeat run is skipped entirely (no model call made).

## Cost Management

This is critical. Heartbeat can be cheap or expensive depending on configuration.

**Token consumption per run:**

| Configuration | Approx. tokens per run |
|---|---|
| Default (main session with history) | ~100K tokens |
| `isolatedSession: true` + `lightContext: true` | ~2-5K tokens |

That's a **20-50x cost reduction** with two config flags.

**Cost reduction strategies (from docs):**

1. **`isolatedSession: true`** — Drops conversation history. Single biggest cost lever. ~100K down to ~2-5K tokens.
2. **`lightContext: true`** — Restricts bootstrap context to only HEARTBEAT.md.
3. **Minimal HEARTBEAT.md** — Less content = fewer tokens.
4. **Cheaper model** — Use `model` override to point heartbeat at a smaller/local model (e.g., Ollama).
5. **`target: "none"`** — Internal-only runs eliminate delivery overhead.
6. **Longer intervals** — 60m = half the cost of 30m.
7. **Active hours** — Prevent runs during off-hours.
8. **Empty HEARTBEAT.md** — Skipped entirely. Zero cost.

**Cron vs Heartbeat cost comparison:**

| Mechanism | Cost Pattern |
|---|---|
| Heartbeat | One turn per interval; scales with checklist size |
| Cron (main session) | Adds event to next heartbeat (no extra turn) |
| Cron (isolated) | Full agent turn per job; use cheaper models |

## Multi-Agent Heartbeat

Per-agent heartbeat schedules are fully supported.

- Different agents can have different intervals, models, targets, and active hours
- If any entry in `agents.list[]` includes a `heartbeat` block, only those agents run heartbeats
- Per-agent settings merge with `agents.defaults.heartbeat`
- An agent without a heartbeat block won't run heartbeats (if any agent has one defined)

**Channel-level scope precedence (highest to lowest):**
1. `channels.<channel>.accounts.<id>.heartbeat`
2. `channels.<channel>.heartbeat`
3. `channels.defaults.heartbeat`
4. `agents.defaults.heartbeat`

**Multi-account delivery example:**
```jsonc
{
  "heartbeat": {
    "target": "telegram",
    "accountId": "ops-bot",
    "to": "12345678:topic:42"  // Thread/topic routing
  }
}
```

## Advanced Patterns

**Event-driven heartbeat:**
- `openclaw system event --text "deployment complete" --mode now` — immediate trigger
- `--mode next-heartbeat` — defers to next scheduled cycle

**Chaining heartbeat with cron:**
Cron jobs in main session mode deposit work items; heartbeat processes them together. Batching pattern.

**Isolated session pattern:**
`isolatedSession: true` + `lightContext: true` + minimal HEARTBEAT.md = cheapest possible monitoring loop (~2-5K tokens). Use for high-frequency, low-context checks.

**Reasoning transparency:**
`includeReasoning: true` sends separate reasoning output. Useful for auditing agent decisions during heartbeat.

**Escalation (encode in HEARTBEAT.md):**
```markdown
- If an alert was sent in the last heartbeat and not acknowledged, escalate to #critical channel
```

**Manual trigger for testing:**
```bash
openclaw system event --text "test" --mode now
```

## Monitoring

- `HEARTBEAT_OK` stripping and drops are logged in standard agent logs
- `useIndicator: true` emits UI status events for Web Control UI
- `includeReasoning: true` delivers reasoning messages for debugging
- Manual trigger available for testing

[Unverified] No dedicated health-check endpoint or dashboard beyond logs and UI indicators.

## Key Takeaways for Workshop

1. Heartbeat is what makes OpenClaw autonomous. Without it, it's just a chatbot.
2. The cost difference between default and optimised config is 20-50x. This is essential knowledge.
3. `isolatedSession` + `lightContext` is the recommended production pattern.
4. Active hours prevent your agent from running (and spending) while you sleep.
5. HEARTBEAT.md is the simplest and most powerful config file in the entire system.
