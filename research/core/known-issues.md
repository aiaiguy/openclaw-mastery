# OpenClaw Known Issues and Workarounds

> Last updated: 2026-03-20 | Source: github.com/openclaw/openclaw/issues

## Project Stats

- 326k GitHub stars, 63.1k forks
- 5,000+ open issues
- 20,667 total commits
- Very active: multiple releases per week
- Responsible disclosure: security@openclaw.ai

## Critical Open Issues (March 2026)

### Gateway status false negative (#51016)
`openclaw status` reports healthy local gateway as unreachable.
**Workaround:** Use `openclaw health --json` or direct HTTP health check.

### Local loopback regression (#51008, v2026.3.13)
CLI commands fail with gateway closure errors (`operator.read failures`).
[Unverified] May require downgrade to previous stable version.

### Multi-agent streaming routing bug (#50976)
In Slack, streaming causes wrong agent to respond in channels bound to specific agents.
**Workaround:** Set `streaming: "off"` for affected Slack channels.

### Agent loop non-termination (#50956)
Queued messages prevent proper task completion, causing full replays.
Related to `messages.queue.mode` setting.

### Model provider switching failure (#50966)
UI model changes do not route requests to the correct provider.
**Workaround:** Use CLI `openclaw models set <provider/model>` instead of UI.

### Local model memory loss (#50894)
Running local models results in complete loss of conversation memory.
No documented workaround.

### Empty memory files (#50752)
Session-memory hook produces empty files on timed-out sessions.
No documented workaround.

### Cron format mismatch (#50942)
Documentation contradicts actual implementation for schedule object format.
**Workaround:** Use string cron expressions instead of objects.

### Telegram polling stall (#50999)
Telegram enters repeated stall/restart loop on macOS.
[Unverified] May be related to network proxy or DNS configuration.

## Common Operational Issues

### SQLite Lock Conflicts
[Unverified] Not explicitly documented as named bug, but session store corruption can occur. The `session.maintenance` settings help. If locks occur: stop gateway, check for stale lock files, restart.

### Token/Connection Disconnections
- `AUTH_TOKEN_MISMATCH`: automatic retry
- Stale device tokens: `openclaw devices clear` and re-pair
- WhatsApp disconnects: `channels.whatsapp.web.reconnect` with exponential backoff

### Rate Limiting
- Anthropic long context: HTTP 429. Reduce `contextTokens` or use compaction more aggressively.
- Gateway auth: 10 failures in 60s triggers 5-minute lockout (configurable).

### Port Conflicts (EADDRINUSE)
Another gateway instance running. Fix: `openclaw gateway stop` then `start`, or change port.

### Model Quirks
- Older models not recommended for tool-enabled or untrusted-input scenarios
- Local models may cause conversation memory loss
- Anthropic long context triggers rate limiting
- Model switching via UI may not route to correct provider

## Diagnostics

```bash
openclaw doctor                  # General health
openclaw doctor --deep           # Thorough audit
openclaw security audit --deep   # Security probe
openclaw channels status --probe # Channel health
openclaw status --deep --all     # Full system status
```
