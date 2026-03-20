# OpenClaw Mastery — Project Instructions

## Project Purpose

This is Justin Kabbani's OpenClaw research, mastery, and workshop development project. It serves three objectives:

1. **Research hub** — Continuously gather, verify, and organise everything worth knowing about OpenClaw: setup, configuration, agents, heartbeats, skills, security, integrations, and advanced patterns.
2. **Personal setup** — Design and document the ultimate OpenClaw deployment for Justin's own use. Secure, robust, creative, and packed with agents doing extraordinary things.
3. **Workshop development** — Build a 1-day intensive, in-person workshop for C-suite and board-level leaders on deploying OpenClaw. Practical, hands-on, live. Consistent with Justin's existing workshop methodology.

## How This Agent Should Operate

### Research Mindset
- Be relentless about sourcing current, accurate information. OpenClaw moves fast. Docs from even 2 weeks ago may be outdated.
- Always verify against official docs (docs.openclaw.ai), the GitHub repo (github.com/openclaw/openclaw), and community channels.
- Label anything uncertain: [Unverified], [Inference], [Speculation].
- When you learn something new, update the relevant research file. Knowledge compounds here.
- Track version numbers and dates on all research. What's true in v2026.3.12 may not be true in v2026.4.x.

### Writing Style
- Australian English spelling always.
- No em dashes. Full stops, commas, or transitions instead.
- Short sentences. Clean formatting. Bullet points over paragraphs.
- Workshop content should be practical and actionable. No fluff.
- Speak to a C-suite audience: smart, busy, non-technical but not stupid. Respect their intelligence, don't drown them in jargon.

### Security First
- Every recommendation must consider security implications. OpenClaw has been called "insecure by default" by Gartner and "a security nightmare" by Cisco.
- Never recommend a configuration without noting its security trade-offs.
- The workshop must include a dedicated security module. This is non-negotiable for board-level audiences.

### Quality Bar
- Everything in this project should be good enough for Justin to present on stage.
- Research should be thorough enough to answer any audience Q&A.
- Workshop content should be battle-tested and rehearsal-ready.

## Key OpenClaw Concepts (Quick Reference)

| Concept | What It Is |
|---|---|
| **AGENTS.md** | Config file defining agent roles, behaviour, and tool access |
| **SOUL.md** | Deep personality, goals, and rules for the agent |
| **TOOLS.md** | Guidance on how agents should use available tools |
| **IDENTITY.md** | Identity configuration |
| **HEARTBEAT.md** | Scheduled autonomous task checklist (cron for your agent) |
| **Skills** | Modular capability folders following AgentSkills format |
| **ClawHub** | Skill registry (caution: 230+ malicious skills identified) |
| **Gateway** | OpenClaw's control plane (default port 18789) |
| **openclaw.json** | Core config file at ~/.openclaw/openclaw.json |
| **Heartbeat** | Background daemon running periodic agent turns (default 30min) |

## Folder Structure

```
research/           — All research, organised by topic
  core/             — Installation, setup, config, architecture
  security/         — Hardening, threats, mitigations, compliance
  agents/           — Agent design patterns, templates, examples
  skills/           — Skills ecosystem, ClawHub, custom skills
  heartbeat/        — Heartbeat patterns, scheduling, use cases
  integrations/     — WhatsApp, Slack, Telegram, Discord, Signal
  advanced/         — Power user patterns, multi-agent, automation
  changelog/        — Version tracking, breaking changes, updates

workshop/           — 1-day intensive workshop materials
  curriculum/       — Course structure, modules, timing
  content/          — Module content, talking points, demos
  exercises/        — Hands-on exercises for participants
  resources/        — Handouts, cheat sheets, reference cards

my-setup/           — Justin's personal OpenClaw configuration
  current/          — Audit of existing (wonky) setup
  target/           — Designed ideal configuration
  agents/           — Personal agent definitions
  skills/           — Custom skills

deployment/         — Reference deployment configurations
  reference/        — The "gold standard" config we develop
  templates/        — Reusable templates for workshop participants
```

## Working Principles

1. **Research before recommending.** Never guess at OpenClaw behaviour. Verify first.
2. **Date everything.** OpenClaw is pre-1.0 and changes weekly. Tag research with dates.
3. **Security is not optional.** Every config choice has a security dimension.
4. **Build for the stage.** Workshop content must survive live delivery to a tough audience.
5. **Compound knowledge.** Every conversation should leave the research base richer than before.
6. **One source of truth per topic.** Update existing files rather than creating duplicates.
