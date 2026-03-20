# OpenClaw Agents — Comprehensive Reference

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai, github.com/mergisi/awesome-openclaw-agents

## The Workspace File Stack

Every agent has a workspace directory containing these files:

| File | Purpose | Injected Into Prompt | Sub-agents Get It? |
|------|---------|---------------------|-------------------|
| AGENTS.md | Operating instructions (what to do) | Every turn | Yes |
| SOUL.md | Personality and persona (who to be) | Every turn | No |
| TOOLS.md | Tool usage guidance | Every turn | Yes |
| IDENTITY.md | Name, emoji, core traits | Every turn | No |
| USER.md | User identity and preferences | Every turn | No |
| MEMORY.md | Long-term curated memory | Private sessions only | No |
| HEARTBEAT.md | Automated check-in checklist | Heartbeat runs | No |
| BOOT.md | Gateway restart checklist | On restart | No |
| BOOTSTRAP.md | First-run setup ritual | First run only | No |

**Key insight for sub-agents:** They only receive AGENTS.md and TOOLS.md. Write those files to be self-contained.

**Size limits:**
- Max per file: 20,000 characters (configurable via `bootstrapMaxChars`)
- Max total across all files: 150,000 characters (`bootstrapTotalMaxChars`)
- Oversized files get truncated with an indicator shown

---

## AGENTS.md

Defines what the agent *does*. Operating procedures, task instructions, workflow sequences, integration rules.

**What goes in AGENTS.md:**
- Operating procedures and rules
- Task-specific instructions ("when the user asks for X, do Y")
- Constraints on output format
- Integration notes (which tools to use, which agents to coordinate with)
- Workflow sequences

**Agent definition in openclaw.json:**
```json5
{
  agents: {
    list: [
      {
        id: "home",
        workspace: "~/.openclaw/workspace-home",
        model: "anthropic/claude-sonnet-4-6",
        // Per-agent sandbox, tool, and elevated overrides
      }
    ]
  }
}
```

---

## SOUL.md

Defines who the agent *is*. Persona, communication tone, personality, behavioural boundaries.

**SOUL.md vs AGENTS.md:**

| SOUL.md | AGENTS.md |
|---------|-----------|
| Who the agent *is* | What the agent *does* |
| Personality and tone | Operating procedures |
| Communication style | Task instructions |
| Behavioural boundaries | Workflow sequences |
| Identity and role | Integration rules |

**Recommended sections (community standard):**

1. **Agent Name and Description** — One-liner on purpose
2. **Core Identity** — Role, Personality, Communication style
3. **Responsibilities** — Numbered list with sub-bullets
4. **Behavioural Guidelines** — "Do:" and "Don't:" sections
5. **Example Interactions** — Concrete dialogue samples
6. **Integration Notes** — Other agents, tool requirements

**Example (Orion, the Coordinator):**
```markdown
# Orion - The Coordinator

You are Orion, an AI coordinator and project manager powered by OpenClaw.

## Core Identity
- **Role:** Task coordinator and project orchestrator
- **Personality:** Professional, efficient, proactive
- **Communication:** Clear, structured, action-oriented

## Responsibilities
1. **Task Management**
   - Break down complex projects into actionable tasks
   - Prioritise work based on urgency and impact
   - Track progress and deadlines

2. **Delegation**
   - Identify the right agent for each task
   - Coordinate multi-agent workflows
   - Ensure smooth handoffs between agents

## Behavioural Guidelines
**Do:**
- Always provide next steps after completing a task
- Ask clarifying questions when requirements are unclear
- Proactively identify potential issues

**Don't:**
- Make assumptions about priorities without asking
- Overwhelm with too many tasks at once
- Skip the summary at the end of complex tasks
```

**Best practices:**
- Be specific in personality descriptions ("Professional, efficient, proactive" beats "helpful and nice")
- Ground guidelines in concrete do's/don'ts, not abstract concepts
- Include realistic example interactions demonstrating the agent's voice
- Keep under 20,000 characters to avoid truncation

---

## TOOLS.md

Informational documentation about local tool conventions. Descriptive guidance, not enforcement.

**Actual tool access control** is in `openclaw.json`:

```json5
{
  tools: {
    allow: ["tool_name", "group:groupname"],
    deny: ["tool_name", "group:groupname"],
    profile: "messaging",  // Restricts to safe messaging tools
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    },
    elevated: {
      enabled: true,
      allowFrom: ["+15551234567"]
    }
  }
}
```

**Available tool groups:**
- `group:automation` — Cron, hooks, webhooks
- `group:runtime` — Process execution, system commands
- `group:fs` — Filesystem read/write
- `sessions_spawn` — Sub-agent spawning
- `sessions_send` — Agent-to-agent messaging
- `gateway` — Config changes, updates
- `cron` — Scheduled job creation
- `memory_search` — Semantic memory recall
- `memory_get` — Targeted memory reads
- `exec`, `read`, `write`, `edit`, `apply_patch`, `process` — Sandboxed tools

---

## IDENTITY.md

Lightweight identity card: agent name, emoji identifier, one-line personality summary.

| IDENTITY.md | SOUL.md |
|-------------|---------|
| Agent name | Full persona |
| Emoji identifier | Communication style |
| Core personality traits (brief) | Detailed behavioural guidelines |
| Machine-readable identity | Human-readable personality |

---

## Multi-Agent Patterns

### Architecture

Each agent is a "fully scoped brain" with its own:
- Workspace (personality files)
- State directory (`~/.openclaw/agents/<agentId>/agent`) for auth and config
- Session store (prevents cross-agent conversation leakage)

**Critical rule:** Never reuse agentDir across agents. Causes auth/session collisions.

### Agent-to-Agent Communication

Off by default. Requires explicit opt-in:
```json5
{
  tools: {
    agentToAgent: {
      enabled: true,
      allow: ["home", "work", "alerts"]
    }
  }
}
```

### Specialisation Patterns

**Channel-based:** WhatsApp routed to fast model for everyday chat. Telegram to capable model for deep work. Discord to coding agent.

**Role-based:** Coordinator manages tasks, delegates to Writer and Analyst. Incident Responder handles alerts, escalates to humans.

**Fleet management:** Central hub coordinating hundreds of agents. 24/7 shift-based swarms on 6-hour rotations. Supervisor agents managing parallel coding instances.

**Sequential and parallel:**
- Chain: Agent A completes, passes to Agent B
- Parallel: Multiple agents work simultaneously, results aggregated
- Sub-agents: Spawned for specific subtasks

---

## Agent Bindings

Bindings route inbound messages to specific agents using channel, account, and peer matching.

```json5
{
  bindings: [
    {
      agentId: "agent-name",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: {
          kind: "direct",       // "direct", "group", or "channel"
          id: "+15551234567"    // Peer identifier
        }
      }
    }
  ]
}
```

**Routing precedence (most specific wins):**
1. Exact peer match (DM/group/channel)
2. Parent peer match (thread inherits)
3. Guild ID + roles (Discord)
4. Guild ID (Discord)
5. Team ID (Slack)
6. Account ID match
7. Channel-level match
8. Default agent fallback

If multiple bindings match same tier, **first in config order wins**.

**WhatsApp (multiple numbers):**
```json5
{
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ]
}
```

**Cross-channel routing (model per channel):**
```json5
{
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },   // Fast model
    { agentId: "opus", match: { channel: "telegram" } }     // Powerful model
  ]
}
```

---

## Trust Levels and Permissions

### Security Model

OpenClaw uses a **personal assistant model** (single trusted operator per gateway). Not multi-tenant.

- One OS user/host per trust boundary
- Separate gateways for mutually untrusted users

### Permission Layers

| Layer | Controls |
|-------|----------|
| `gateway.auth` | API caller authentication |
| `dmPolicy` | Who can message the bot |
| `tools.allow/deny` | What tools agents can use |
| `sandbox.mode` | Where tools execute |
| `elevated` | Host-level execution escape |
| File permissions | Config integrity |

### DM Policies (Inbound Access)

- **`pairing`** (default): Unknown senders get expiring codes. 3 pending max.
- **`allowlist`**: Block unknown senders entirely.
- **`open`**: Requires explicit `allowFrom: ["*"]`.
- **`disabled`**: Ignore all inbound DMs.

### Hardened Baseline (Recommended Starting Point)

```json5
{
  gateway: { mode: "local" },
  channels: { whatsapp: { dmPolicy: "pairing" } },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs",
           "sessions_spawn", "sessions_send", "gateway", "cron"]
  },
  agents: {
    defaults: {
      sandbox: { mode: "non-main" },
      elevatedDefault: "off"
    }
  }
}
```

### Graduated Agent Profiles

| Profile | Sandbox | Tools | Use Case |
|---------|---------|-------|----------|
| Personal | Off | Full access | Trusted operator's own agent |
| Family | On | Read-only | Shared household agent |
| Public | On | No filesystem/shell | Open-access bot |

### Elevated Mode

Four layers of gating (all must pass):
1. Global feature gate: `tools.elevated.enabled`
2. Global sender allowlist: `tools.elevated.allowFrom`
3. Per-agent gate: `agents.list[].tools.elevated.enabled`
4. Per-agent allowlist

Three states: `off` (default), `on`/`ask` (with approvals), `full` (auto-approve).

### Critical Controls to Deny by Default

- `gateway` tool (can modify config, run updates)
- `cron` tool (creates persistent scheduled jobs)
- `group:automation`, `group:runtime`, `group:fs`
- `sessions_spawn`, `sessions_send` (agent-to-agent)

### File Permissions

- `~/.openclaw/openclaw.json`: `600` (user read/write only)
- `~/.openclaw/`: `700` (user-only directory access)

---

## Community Agent Templates

The awesome-openclaw-agents repo has **177 production-ready templates** across **24 categories**.

| Category | Count | Standout Templates |
|----------|-------|--------------------|
| Marketing and Content | 17 | Echo (writer), Buzz (social), Book Writer |
| Business | 12 | Pipeline (sales), Sentinel (churn risk) |
| DevOps | 11 | Incident Responder, Self-Healing Server |
| Development | 10 | Lens (PR review), Trace (error analysis) |
| Finance | 10 | Trading Bot, Fraud Detector |
| Productivity | 7 | Orion (coordinator), Pulse (analytics) |
| Personal | 7 | Atlas (schedule), Iron (fitness) |
| Education | 4 | Tutor, Quiz Maker, Research Assistant |

Also: Healthcare, Legal, HR, Creative, Security, E-Commerce, Data, SaaS, Real Estate, Freelance, Supply Chain, Compliance, Voice, Customer Success, Automation.

**Quick start:**
```bash
git clone https://github.com/mergisi/awesome-openclaw-agents.git
cd awesome-openclaw-agents/quickstart
npm install && cp ../agents/productivity/orion/SOUL.md ./SOUL.md
node bot.js
```

---

## Agent Design Mistakes to Avoid

1. **Reusing agentDir across agents** — Auth/session collisions
2. **SOUL.md too long** — Truncated at 20,000 chars, losing critical instructions at the end
3. **Mixing identity and operations** — SOUL.md for personality, AGENTS.md for procedures
4. **Over-permissioning tools** — Start deny-by-default, add what's needed
5. **Skipping example interactions** — Model performs better with concrete demonstrations
6. **Using weak models with tool access** — "Older/smaller/legacy models are significantly less robust against prompt injection and tool misuse"

---

## Key Takeaways for Workshop

1. The file stack is the configuration system. No GUI, no database. Just Markdown files.
2. SOUL.md is the "soul" of your agent. Get this right and everything else follows.
3. Start with the hardened baseline config and open up selectively.
4. Bindings give you powerful routing. One phone number, many agents.
5. 177 community templates means nobody starts from scratch.
6. Tool permissions are the security boundary. Deny by default.
