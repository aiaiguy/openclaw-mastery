# OpenClaw Advanced Patterns and Automation

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai, github.com/hesamsheikh/awesome-openclaw-usecases

## Three Automation Primitives

### 1. Hooks (Event-Driven, In-Gateway)

Respond to agent events with TypeScript functions.

**Trigger points:**
- Commands (`/new`, `/reset`, `/stop`)
- Session lifecycle (compaction, bootstrap)
- Messages (received/sent/transcribed)
- Gateway startup

**Examples:**
- Session memory capture on `/new`
- Command audit logging to JSONL
- Loading monorepo config at bootstrap
- Executing `BOOT.md` at gateway startup

### 2. Cron Jobs (Scheduled)

Three schedule types: one-shot (`at`), interval (`every`), cron expression (5/6-field + timezone).

Two execution models:
- **Main session** — Adds to next heartbeat turn (no extra API call)
- **Isolated session** — Dedicated agent turn (independent cost)

**Key features:**
- Persistent custom sessions maintain context across runs (daily standups that reference yesterday's summary)
- Delivery modes: announce (to channel), webhook (HTTP POST), or none
- Retry with exponential backoff
- Model override per job

**Example:**
```bash
openclaw cron add --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "Australia/Melbourne" \
  --session isolated \
  --message "Summarise overnight updates." \
  --announce --channel slack
```

### 3. Webhooks (External Triggers)

HTTP endpoints accepting payloads routed to agents.
- Token-based authentication
- Gmail Pub/Sub integration for inbound email triggers
- Payload mappings route specific webhooks to specific agents
- Treat payloads as untrusted

---

## Browser Automation

Dedicated, isolated browser profile using Chrome DevTools Protocol with Playwright.

**Capabilities:**
- Navigate URLs (SSRF-guarded)
- AI snapshots with numeric refs or ARIA role-based snapshots
- Screenshots, PDF export
- Click, double-click, drag, text input, form filling, select, hover, scroll, keyboard
- Cookie get/set/clear, localStorage/sessionStorage
- Geolocation, timezone, locale, device simulation, dark/light mode
- Console/error/network inspection, JavaScript evaluation, tracing
- File downloads and uploads
- Wait conditions: URL patterns, load states, JS predicates, selector visibility

**Browser profiles:**
- Local managed Chromium (default)
- Remote CDP endpoints
- Existing browser sessions (Chrome, Brave, Edge) reusing logged-in state
- Hosted: Browserless, Browserbase

**Limitations:**
- Playwright required for navigate, act, AI snapshot, element screenshots, PDF
- `evaluate` executes arbitrary page JS. Disable with `browser.evaluateEnabled=false` if prompt injection is a concern

---

## File and Document Processing

| Type | Capability |
|------|-----------|
| Images | Analyse via configured image models. Generate with `image_generate`. |
| Audio | Automatic transcription hooks |
| PDF | Dedicated `pdf` tool |
| Documents | Media handling across channels |
| Video | Camera snapshots/clips from connected nodes |

Vision: `imageMaxDimensionPx` (default 1200) controls token optimisation.

---

## Calendar and Scheduling

No native calendar channel. Handled through:
- **Cron jobs** for scheduling within OpenClaw
- **65+ Calendar/Scheduling skills** on ClawHub
- **Browser automation** for calendar web UIs
- **Composio** for calendar API bridges

---

## CRM and Business Tool Integration

Not native. Well-supported through:
- **Composio integration** — managed OAuth across 1,000+ apps (HubSpot, Salesforce, Notion, Linear, etc.)
- **ActiveCampaign CRM** skill for lead/deal management
- **Airtable** via Rube MCP (Composio)
- Community: "Local CRM Framework" using DuckDB + browser automation
- **105+ Marketing and Sales skills** in ecosystem

---

## Custom API Integration

**Four approaches:**

1. **Skills** — Write SKILL.md teaching agent to call APIs via `exec` (curl) or `web_fetch`
2. **Composio** — Managed OAuth, scoped permissions, logged tool calls across 1,000+ apps. Path of least resistance.
3. **Webhooks** — Inbound API triggers. External services POST to OpenClaw endpoints.
4. **n8n / Automation Platforms** — Agent calls n8n via webhooks; n8n handles credentials and API calls. Agent never touches raw keys. Community favourite pattern.

**Authentication patterns:**
- Environment variables for API keys
- Secret references: `env`, `file`, or `exec` providers
- Composio handles OAuth flows automatically
- Webhook tokens for inbound auth

---

## Performance Optimisation

### Token Reduction
- `imageMaxDimensionPx` (default 1200) resizes images before model
- Session pruning trims old tool results from context automatically
- `/compact` manually summarises older context
- `lightContext: true` for cron jobs that don't need workspace files
- Pre-compaction memory flush prompts model to write durable notes
- Skills: ~24 tokens per skill. Disable unused ones.

### Response Speed
- Model fallbacks: `agents.defaults.model.fallbacks` for automatic failover
- Streaming: `partial` mode shows progressive output
- Background execution: `exec` tool supports background processes

### Session Management
- Daily or idle resets prevent unbounded context growth
- `sessionRetention` for cron (default 24h) prunes completed sessions
- `maxEntries` caps session count (default 500)
- `maxDiskBytes` + `highWaterBytes` for storage limits
- Run-log trimming: `maxBytes` (2MB) and `keepLines` (2000)

### Hot Reload
Config changes apply without downtime in `hybrid` mode (default). Only gateway server changes require restart.

### Health Monitoring
- `channelHealthCheckMinutes` (default 5)
- `channelStaleEventThresholdMinutes` (default 30)
- `channelMaxRestartsPerHour` (default 10)
- Auto-restarts unhealthy channels

---

## Community Power User Tips

- **STATE.yaml pattern** — For subagent coordination without orchestrator overhead. Agents read/write shared YAML state file.
- **Webhook credential isolation** — Use n8n to handle API credentials. Agent triggers webhooks; n8n does authenticated calls.
- **Persistent cron sessions** — `session:custom-id` for jobs needing memory across runs (daily standups referencing yesterday).
- **Identity linking** — Map same person across WhatsApp, Telegram, Discord to single session via `session.identityLinks`.
- **Multi-agent teams via Telegram** — Specialised agents bound to different groups or topics.
- **Pre-Build Validator skill** — Before building anything, scan GitHub, npm, PyPI to check if it already exists.
- **`/think` levels** — Adjust reasoning depth per turn for cost control.
- **`/verbose on|off`** — Toggle output detail.
- **`/usage`** — Monitor token spend in real-time.
- **`openclaw doctor --fix`** — Diagnose and auto-repair configuration issues.
- **`$include`** — Split large configs into multiple files with deep merging.
- **Sticker cache** — Telegram sticker descriptions cached locally to avoid repeated vision model calls.
- **Execution approvals** — Route dangerous tool executions through approval buttons before they run.
- **Mobile nodes** — iOS/Android devices as nodes. Camera, screen recording, voice from phone.
- **Config writes from chat** — `/config set` and `/config unset` update live (disable with `configWrites: false`).

---

## What NOT to Do

1. **Do not use your personal Signal number.** Can de-authenticate your main Signal app.
2. **Do not set `dmPolicy: "open"` without understanding risk.** Use pairing (default) or allowlists.
3. **Do not use Bun as runtime.** Incompatible. Stick to Node.js.
4. **Do not hardcode tokens in config.** Use env vars, `.env`, or secret references.
5. **Do not enable `allowBots=true` on Discord without mention/allowlist rules.** Bot loop risk.
6. **Do not enable `browser.evaluateEnabled` in untrusted contexts.** Prompt injection vector.
7. **Do not skip reviewing third-party skills.** Treat as untrusted code.
8. **Do not manually edit `~/.openclaw/cron/jobs.json` while gateway running.** Gets overwritten.
9. **Do not use default `dmScope: "main"` for multi-user.** Shares one session across all DMs. Use `per-channel-peer`.
10. **Do not forget Telegram Privacy Mode.** Bot won't see non-mentioned messages unless disabled and bot re-added.
11. **Do not neglect session maintenance.** Set `pruneAfter`, `maxEntries`, `maxDiskBytes` in production.
12. **Do not use `replyToMode='off'` in Slack if you want threading.** Disables all threading including explicit tags.

---

## Real-World Automation Examples (Community)

- Multi-channel customer service (WhatsApp + email + Google Reviews unified inbox)
- Self-healing home server with SSH + cron
- n8n workflow orchestration for credential-safe API calls
- Morning briefing with news, tasks, content drafts
- Event confirmation via AI voice calls
- Multi-agent content factory: Discord pipeline with research, writing, and thumbnail agents
- Family calendar assistant aggregating multiple calendars
- Personal CRM auto-discovering contacts from email and calendar

---

## Key Takeaways for Workshop

1. Three primitives (hooks, cron, webhooks) cover most automation needs.
2. Browser automation is production-grade. Real form filling, real logins, real scraping.
3. Composio is the shortcut for SaaS integrations. 1,000+ apps with managed OAuth.
4. The n8n credential isolation pattern is a security best practice worth teaching.
5. Session management is the thing people forget. Unbounded sessions = unbounded cost.
6. The "what NOT to do" list is workshop gold. Saves people from real pain.
