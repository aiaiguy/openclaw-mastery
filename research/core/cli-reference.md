# OpenClaw CLI Reference

> Last updated: 2026-03-20 | Source: docs.openclaw.ai/cli

## Setup and Configuration

| Command | Purpose |
|---------|---------|
| `openclaw setup` | Initialise configuration and workspace |
| `openclaw onboard` | Interactive onboarding wizard |
| `openclaw configure` | Interactive config wizard (models, channels, skills, gateway) |
| `openclaw config get\|set\|unset\|file\|validate` | Non-interactive config management |
| `openclaw doctor` | Health checks and quick fixes (`--deep` for thorough audit) |

## Gateway Management

| Command | Purpose |
|---------|---------|
| `openclaw gateway` | Run WebSocket gateway |
| `openclaw gateway status` | Probe health |
| `openclaw gateway install\|uninstall\|start\|stop\|restart` | Service lifecycle |
| `openclaw gateway call <method>` | Direct RPC calls |
| `openclaw logs` | Tail gateway logs (`--follow`, `--limit`, `--json`) |

## Agent and Messaging

| Command | Purpose |
|---------|---------|
| `openclaw agent --message <text>` | Run one agent turn |
| `openclaw agents list\|add\|bindings\|bind\|unbind\|delete` | Manage agents |
| `openclaw message send\|poll\|react\|read\|edit\|delete\|pin\|thread` | Unified messaging |
| `openclaw sessions` | List conversation sessions |
| `openclaw pairing list\|approve` | DM pairing management |

## Channels

| Command | Purpose |
|---------|---------|
| `openclaw channels list\|status\|logs\|add\|remove\|login\|logout` | Channel management |

## Models

| Command | Purpose |
|---------|---------|
| `openclaw models list` | List available models |
| `openclaw models list --all --json` | All models, JSON output |
| `openclaw models status --probe` | Status with live capability check |
| `openclaw models set <provider/model>` | Set primary model |
| `openclaw models set-image <provider/model>` | Set image model |
| `openclaw models aliases list\|add\|remove` | Model aliases |
| `openclaw models fallbacks list\|add\|remove\|clear` | Fallback chain |
| `openclaw models scan` | Auto-discover via OpenRouter |
| `openclaw models auth add` | Add auth profile |
| `openclaw models auth order get\|set\|clear` | Provider priority |

## Nodes and Devices

| Command | Purpose |
|---------|---------|
| `openclaw node run\|status\|install\|uninstall\|stop\|restart` | Headless node host |
| `openclaw nodes status\|list\|pending\|approve\|reject\|describe\|invoke\|run\|notify\|camera\|canvas\|location` | Gateway-paired nodes |
| `openclaw devices list\|approve\|reject\|remove\|clear\|rotate\|revoke` | Device pairing |

## Tools and Automation

| Command | Purpose |
|---------|---------|
| `openclaw skills list\|info\|check` | Skill management |
| `openclaw plugins list\|inspect\|install\|enable\|disable\|doctor` | Plugin management |
| `openclaw cron status\|list\|add\|edit\|rm\|enable\|disable\|runs\|run` | Scheduled jobs |
| `openclaw hooks list\|info\|check\|enable\|disable\|install\|update` | Automation hooks |
| `openclaw webhooks gmail setup\|run` | Gmail integration |
| `openclaw browser` | Chrome control |

## System

| Command | Purpose |
|---------|---------|
| `openclaw system event --text <text>` | Enqueue system event |
| `openclaw system heartbeat last\|enable\|disable` | Heartbeat controls |
| `openclaw system presence` | List presence entries |
| `openclaw memory status\|index\|search` | Vector search over memory |

## Security and Maintenance

| Command | Purpose |
|---------|---------|
| `openclaw secrets reload\|configure\|apply` | Credential management |
| `openclaw security audit` | Security auditing (`--deep`, `--fix`) |
| `openclaw backup create\|verify` | State backup |
| `openclaw reset` | Reset config/state |
| `openclaw uninstall` | Remove OpenClaw |
| `openclaw update` | Update (`--channel stable\|beta\|dev`) |

## Utilities

| Command | Purpose |
|---------|---------|
| `openclaw dashboard` | Open web dashboard |
| `openclaw tui` | Terminal UI |
| `openclaw docs [query...]` | Search live documentation |
| `openclaw completion` | Shell completion setup |
| `openclaw health` | Gateway health check |

## Global Flags

- `--dev` — Isolate state under `~/.openclaw-dev`
- `--profile <name>` — Isolate state under `~/.openclaw-<name>`
- `--no-color` — Disable ANSI colours
- `--version` / `-V` — Print version

## Chat Commands (In-Platform)

| Command | Purpose |
|---------|---------|
| `/status` | Session status |
| `/new` or `/reset` | Reset session |
| `/compact` | Compress context |
| `/think <level>` | Set thinking depth |
| `/verbose on\|off` | Toggle verbosity |
| `/usage off\|tokens\|full` | Usage reporting |
| `/restart` | Restart gateway |
| `/activation mention\|always` | Group mode toggle |
| `/model` | Switch model |
| `/elevated on\|off` | Toggle elevated bash |
| `/config set\|unset` | Live config changes |
