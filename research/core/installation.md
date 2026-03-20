# OpenClaw Installation and Setup

> Last updated: 2026-03-20 | Version: v2026.3.13-1

## Requirements

- Node 22 or newer

## Install

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

## Workspace Structure

Default location: `~/.openclaw/workspace/`

| File | Purpose |
|---|---|
| `AGENTS.md` | Agent definitions, behaviour, instructions |
| `SOUL.md` | Deep personality traits, long-term goals, rules |
| `TOOLS.md` | Guidance on tool usage |
| `IDENTITY.md` | Identity configuration |
| `HEARTBEAT.md` | Routine task checklist (plain English) |
| `skills/<name>/SKILL.md` | Individual skill definitions |

## Core Config

`~/.openclaw/openclaw.json`

Requires Gateway restart or `config.patch` RPC to apply changes.

## Gateway

OpenClaw's control plane. Default port: **18789**.

## Key Config Options

| Setting | What It Controls |
|---|---|
| `agents.defaults.heartbeat.every` | Heartbeat interval (default 30m) |
| `agents.defaults.heartbeat.prompt` | Custom heartbeat prompt |
| `agents.defaults.workspace` | Workspace directory path |
| `agents.list[]` | Per-agent configurations |
| Heartbeat target | `"none"`, `"last"`, or specific channel |
| Active hours + timezone | When heartbeat can run |
| Gateway port | Default 18789 |

## TODO: Research Gaps

- [ ] Full `openclaw.json` schema documentation
- [ ] Docker installation method
- [ ] Multi-workspace setup patterns
- [ ] Upgrade path between versions
- [ ] Backup and restore procedures
