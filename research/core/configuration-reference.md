# OpenClaw Configuration Reference

> Last updated: 2026-03-20 | Source: docs.openclaw.ai/gateway/configuration-reference

## Config File

Location: `~/.openclaw/openclaw.json` (JSON5 format, supports comments and trailing commas). All fields optional with safe defaults.

## Top-Level Sections

| Section | Purpose |
|---------|---------|
| `gateway` | Server binding, auth, networking, hot reload |
| `channels` | Communication platform configs |
| `agents` | Agent definitions, model, workspace, sandbox, heartbeat |
| `session` | Conversation scope, lifecycle, identity links |
| `messages` | Response formatting, queue behaviour, TTS |
| `tools` | Capability profiles, exec policies, loop detection, browser, media |
| `models` | LLM provider definitions, custom proxies, Bedrock discovery |
| `browser` | Chrome/Chromium automation, SSRF policies |
| `commands` | Slash command configuration |
| `skills` | Bundled/managed/workspace skill config |
| `plugins` | Extension loading, hooks, slot overrides |
| `ui` | Visual theming, assistant name/avatar |
| `talk` | Voice/TTS configuration |

---

## Gateway Configuration

```json5
{
  gateway: {
    mode: "local",              // "local" | "remote"
    port: 18789,                // Default port
    bind: "loopback",           // "loopback" | "lan" | "tailnet" | "custom"
    auth: {
      mode: "token",            // "none" | "token" | "password" | "trusted-proxy"
      token: "your-token",
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off",              // "off" | "serve" | "funnel"
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
    },
  },
}
```

**Port resolution:** `--port` flag > `OPENCLAW_GATEWAY_PORT` env > config > fallback 18789

**Hot reload modes:** `off` | `hot` (safe only) | `restart` (full) | `hybrid` (default, smart combo)

---

## Agent Defaults

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      bootstrapMaxChars: 20000,          // Per-file truncation
      bootstrapTotalMaxChars: 150000,    // Total budget
      imageMaxDimensionPx: 1200,
      userTimezone: "Australia/Melbourne",
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.7"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
      },
      imageGenerationModel: { primary: "openai/gpt-image-1" },
      pdfModel: { primary: "anthropic/claude-opus-4-6" },
      thinkingDefault: "low",            // "off"|"minimal"|"low"|"medium"|"high"
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      contextTokens: 200000,
      maxConcurrent: 3,
      heartbeat: { /* see heartbeat docs */ },
      compaction: {
        mode: "default",
        timeoutSeconds: 900,
        reserveTokensFloor: 24000,
        model: "openrouter/anthropic/claude-sonnet-4-6",
        memoryFlush: { enabled: true, softThresholdTokens: 6000 },
      },
      contextPruning: {
        mode: "cache-ttl",
        ttl: "1h",
        keepLastAssistants: 3,
      },
      sandbox: {
        mode: "off",                     // "off"|"non-main"|"all"
        backend: "docker",
        scope: "agent",                  // "agent"|"session"|"shared"
      },
    },
  },
}
```

---

## Session Configuration

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",                     // "main"|"per-peer"|"per-channel-peer"|"per-account-channel-peer"
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",                     // "daily"|"idle"
      atHour: 4,
      idleMinutes: 60,
    },
    maintenance: {
      mode: "warn",                      // "warn"|"enforce"
      pruneAfter: "30d",
      maxEntries: 500,
      maxDiskBytes: "500mb",
    },
  },
}
```

---

## Tools Configuration

```json5
{
  tools: {
    profile: "coding",                   // "minimal"|"coding"|"messaging"|"full"
    allow: ["browser"],
    deny: ["canvas"],
    elevated: { enabled: true, allowFrom: { whatsapp: ["+15555550123"] } },
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
    },
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
    },
    web: {
      search: { enabled: true, apiKey: "brave_api_key", maxResults: 5 },
      fetch: { enabled: true, maxChars: 50000 },
    },
    subagents: {
      model: "minimax/MiniMax-M2.7",
      maxConcurrent: 1,
    },
  },
}
```

---

## Model Configuration

### Supported Providers

- Anthropic (Claude Opus, Sonnet, Haiku)
- OpenAI (GPT-4.1, GPT-5.2, o-series)
- Google (Gemini)
- Mistral
- OpenRouter (100+ models)
- MiniMax (MiniMax-M2.7)
- Local models (custom provider with baseUrl)
- AWS Bedrock (auto-discovery)
- Custom proxies (LiteLLM, etc.)

### Model Selection Hierarchy

Three-tier failover:
1. Primary model attempted first
2. Auth profile rotation within same model
3. Fallback models tried in order

### Per-Agent and Per-Channel Overrides

```json5
{
  agents: { list: [
    { id: "main", model: "anthropic/claude-opus-4-6" },
    { id: "cheap", model: "openai/gpt-4.1-mini" },
  ]},
  channels: {
    modelByChannel: {
      discord: { "123456789": "anthropic/claude-opus-4-6" },
      slack: { "C1234567890": "openai/gpt-4.1" },
    },
  },
}
```

### Custom Provider (e.g., LiteLLM)

```json5
{
  models: {
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [{
          id: "llama-3.1-8b",
          contextWindow: 128000,
          maxTokens: 32000,
        }],
      },
    },
  },
}
```

### Built-in Aliases

`opus`, `sonnet`, `gpt`, `gpt-mini`, `gemini`, `gemini-flash`, `gemini-flash-lite`

### Runtime Model Switching

- `/model` or `/model list` — interactive picker
- `/model <provider/model>` — direct selection
- `/model status` — show current config

---

## Secrets Management

### Three Ways to Provide Secrets

1. **Plain text** (not recommended): `"sk-abc123"`
2. **Environment variable:** `"${ANTHROPIC_API_KEY}"`
3. **SecretRef:**
```json5
{ source: "env", provider: "default", id: "ANTHROPIC_API_KEY" }
{ source: "file", provider: "default", id: "/path/to/secret" }
{ source: "exec", provider: "default", id: "op read op://vault/item/field" }
```

### Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway bearer token |
| `OPENCLAW_GATEWAY_PASSWORD` | Gateway password auth |
| `OPENCLAW_GATEWAY_PORT` | Override default port |
| `OPENCLAW_CONFIG_PATH` | Custom config file path |
| `OPENCLAW_STATE_DIR` | Custom state directory |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |

### Log Redaction

- `logging.redactSensitive: "tools"` (default) redacts tool summaries
- `logging.redactPatterns` accepts custom regex for environment-specific tokens

### CLI

```bash
openclaw secrets reload          # Re-resolve refs
openclaw secrets configure       # Interactive setup
openclaw security audit --fix    # Auto-fix permissions
```

---

## Backup and Restore

### Built-in

```bash
openclaw backup create           # Create backup
openclaw backup verify           # Verify integrity
```

### Git-Based Workspace Backup (Recommended)

Private git repo in workspace directory. Include: AGENTS.md, SOUL.md, TOOLS.md, IDENTITY.md, USER.md, HEARTBEAT.md, MEMORY.md, memory/. Exclude: credentials, session transcripts.

### Migration Between Machines

1. Clone workspace git repo on new machine
2. Install OpenClaw
3. Run setup: `openclaw setup --workspace <path>`
4. Re-run onboarding: `openclaw onboard`
5. Copy `openclaw.json` (redact secrets or use env var substitution)

### Full State

Complete state under `~/.openclaw/`. For full migration: copy directory, install OpenClaw, run `openclaw doctor`.
