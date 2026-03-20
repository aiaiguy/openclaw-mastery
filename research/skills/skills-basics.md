# OpenClaw Skills — Comprehensive Reference

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai/skills, docs.openclaw.ai/tools/skills-config.md, docs.openclaw.ai/tools/creating-skills.md, clawhub.ai

## What Skills Are

Modular capability folders following the AgentSkills format. Each skill is a directory containing a `SKILL.md` with YAML frontmatter and Markdown instructions. Skills extend what your agent can do without modifying core config.

## SKILL.md Specification

**YAML frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier for the skill |
| `description` | Yes | Brief explanation of functionality |
| `homepage` | No | URL displayed in macOS UI |
| `user-invocable` | No | Boolean, default `true`. Exposes as slash command |
| `disable-model-invocation` | No | Excludes from model prompts but allows user invocation |
| `command-dispatch` | No | Set to `"tool"` to bypass model and dispatch directly to tools |
| `command-tool` | No | Specifies tool name for dispatch mode |
| `metadata` | No | Single-line JSON object for gating and configuration |

**Important:** The embedded agent parser supports single-line frontmatter keys only.

**Metadata gating system** (within `metadata.openclaw`):

```json
{
  "requires": {
    "bins": ["required-binary"],
    "anyBins": ["at-least-one-of-these"],
    "env": ["REQUIRED_ENV_VAR"],
    "config": ["openclaw.json-paths"]
  },
  "primaryEnv": "API_KEY_NAME",
  "os": ["darwin", "linux", "win32"],
  "always": true,
  "emoji": "icon",
  "homepage": "url"
}
```

Environment variables must exist or be provided in config. Binary requirements checked at load time.

## Skill Loading and Execution

- Skills are snapshotted when a session starts and reused for subsequent turns
- Mid-session refresh occurs through the skills watcher or new remote nodes
- Changes take effect on the next new session

**Token cost of skills in system prompt:**
- Base overhead: 195 characters (when at least one skill present)
- Per skill: 97 + len(name) + len(description) + len(location) characters
- Rough estimate: ~4 chars/token, so ~24 tokens per skill base overhead

## Skill Locations (Priority Order)

1. `<workspace>/skills/` — Workspace skills, highest priority
2. `~/.openclaw/skills/` — Managed/local skills, middle priority
3. Bundled skills — Shipped with install, lowest priority

Additional directories via `skills.load.extraDirs` in `openclaw.json`.

## Writing Custom Skills

**Step 1: Create the directory**
```bash
mkdir -p ~/.openclaw/workspace/skills/my-skill
```

**Step 2: Write SKILL.md**
```markdown
---
name: my_skill
description: A brief description of what this skill does.
---

# My Skill

When the user asks for [trigger condition], do the following:

1. Use the `bash` tool to run [command]
2. Parse the output for [specific data]
3. Format and return the result
```

**Step 3: Tool integration**
Define custom tools in frontmatter or direct the agent to use existing system tools (bash, browser, etc.).

**Step 4: Activation**
Request the agent to "refresh skills" or restart the gateway.

**Best practices from docs:**
- "Instruct the model on what to do, not how to be an AI"
- "Ensure the prompts don't allow arbitrary command injection from untrusted user input"
- Test locally: `openclaw agent --message "use my new skill"` before deployment

## Skill Filtering and Trust

**Per-agent isolation:**
- Each agent has its own workspace. Per-agent skills live in `<workspace>/skills/`.
- Shared skills in `~/.openclaw/skills/` are accessible across all agents.

**Configuration-based filtering:**
```jsonc
{
  "skills": {
    "entries": {
      "skill-name": {
        "enabled": false  // Disables even bundled skills
      }
    },
    "allowBundled": ["whitelist-these-only"]  // Allowlist for bundled skills only
  }
}
```

- `enabled: false` disables any skill (bundled, managed, or workspace)
- `allowBundled` creates a whitelist for bundled skills. Managed and workspace skills unaffected.

**Environment/secret injection per skill:**
```jsonc
{
  "skills": {
    "entries": {
      "my-skill": {
        "env": { "MY_VAR": "value" },
        "apiKey": { "source": "env", "provider": "default", "id": "KEY_NAME" }
      }
    }
  }
}
```

Injected into `process.env` at agent run start, restored after.

**Sandboxing:** Recommended for untrusted inputs. Binary requirements in sandboxed agents need binaries installed via `setupCommand`.

[Unverified] No explicit trust-level system (e.g., trusted/untrusted/verified tiers). Trust is managed through manual review, workspace isolation, and config toggles.

## ClawHub

**URL:** clawhub.ai

ClawHub is OpenClaw's public skill registry. Self-described as "the skill dock for sharp agents."

**How it works:**
- Users publish skill bundles (directories with SKILL.md + supporting files)
- Skills versioned like npm packages with changelog and rollback
- Vector-powered search beyond basic keyword matching
- Community signals: stars and comments

**CLI commands:**
```bash
npx clawhub@latest install <skill-slug>
clawhub search "postgres backups"
clawhub update --all
clawhub sync --all                  # Scan and publish updates
```

Installed skills tracked in `.clawhub/lock.json`. Default to workspace installation.

**Vetting process:**
- New publishers need GitHub account at least 1 week old
- Community moderation: any user can report problematic skills
- Skills with 3+ unique reports are hidden by default
- Moderators retain override authority

**Current state (March 2026):** Early-adopter phase. No prominently featured or highly-downloaded skills yet. Platform is open source (MIT), deployed on Vercel, powered by Convex.

**Security warning from docs:** "Treat third-party skills as untrusted code. Read them before enabling." No automated code scanning at registry level.

## Modular Memory

Memory operates through a **plugin slot architecture**. Default plugin is `memory-core`, disableable via `plugins.slots.memory = "none"`.

**Two-layer file structure:**
1. **Daily logs** (`memory/YYYY-MM-DD.md`) — append-only, session-specific notes
2. **Curated long-term** (`MEMORY.md` or `memory.md`) — persistent knowledge, loaded in private sessions only

The docs state: "The files are the source of truth; the model only 'remembers' what gets written to disk."

**Agent-facing tools:**
- `memory_search` — semantic retrieval across indexed snippets (~400 token chunks, 80-token overlap)
- `memory_get` — targeted file/line-range reads (paths outside MEMORY.md/memory/ rejected)

**Search configuration (`agents.defaults.memorySearch`):**
- **Embedding providers** (auto-selected): local, openai, gemini, voyage, mistral
- **Hybrid search:** BM25 + vector matching with configurable weights
- **MMR (Maximal Marginal Relevance):** diversity re-ranking, lambda 0.0-1.0, default 0.7
- **Temporal decay:** recency boosting, default 30-day half-life
- **Multimodal memory** (Gemini only): indexes images and audio from `extraPaths`
- **Local embeddings:** default model `embeddinggemma-300m` (~0.6 GB, auto-downloads)

**Index storage:** SQLite at `~/.openclaw/memory/<agentId>.sqlite` with optional sqlite-vec acceleration.

**QMD backend (experimental):** Alternative to SQLite using Bun-powered sidecar with node-llama-cpp for fully local operation.

## MCP (Model Context Protocol) Integration

[Unverified] OpenClaw does not appear to natively support MCP servers as of current documentation. The tool integration model is plugin-based with its own registration system. If MCP support exists, it is not documented in official docs.

## Community Skills and Plugins

**Verified community plugins (from docs):**
1. **openclaw-dingtalk** — DingTalk enterprise robot integration (Stream mode)
2. **QQbot** — QQ connectivity with private chats, group mentions, rich media
3. **WeChat** — WeChat personal accounts via iPad protocol

**Submission requirements:**
- Published on npmjs
- Hosted in public GitHub repos
- Include setup docs and issue tracking
- Active maintenance

## Key Takeaways for Workshop

1. Skills are the extensibility mechanism. They're how you make OpenClaw do specific things well.
2. Custom skills are just Markdown files. No coding required for basic skills.
3. ClawHub is useful but dangerous. Vet everything. Read the SKILL.md before installing.
4. The cost of skills in the system prompt is small but compounds. Only load what you need.
5. Memory is local-first and file-based. The agent only remembers what's written to disk.
6. `isolatedSession` + `lightContext` (from heartbeat) also affects which skills are loaded.
