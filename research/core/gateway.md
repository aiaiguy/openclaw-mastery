# OpenClaw Gateway

> Last updated: 2026-03-20 | Source: docs.openclaw.ai/gateway

## What It Is

Single, always-on daemon process. The control plane for all OpenClaw operations:
- Routes messages between chat platforms and AI agents
- Manages sessions, tools, and event streaming
- Serves Control UI and WebChat
- Handles WebSocket connections from CLI, macOS app, iOS/Android nodes
- Multiplexes WebSocket, HTTP API (OpenAI-compatible), and Control UI on a single port

## Defaults

- Port: `18789`
- Bind: `loopback` (127.0.0.1 only)
- Control UI: `http://127.0.0.1:18789/`
- WebSocket: `ws://127.0.0.1:18789`

## Health Checks

```bash
curl -fsS http://127.0.0.1:18789/healthz    # Liveness
curl -fsS http://127.0.0.1:18789/readyz      # Readiness
openclaw health --json                        # CLI
openclaw gateway status --deep                # Deep probe
openclaw channels status --probe              # Channel health
```

WebSocket liveness: send `connect` frame, expect `hello-ok` response with presence, health, state version, uptime, limits.

## Service Management

```bash
openclaw gateway install      # Install as system service
openclaw gateway uninstall    # Remove service
openclaw gateway start        # Start
openclaw gateway stop         # Stop
openclaw gateway restart      # Restart
openclaw gateway status       # Check status
```

## Logging

```bash
openclaw logs --follow        # Tail live
openclaw logs --limit 100     # Last 100 lines
openclaw logs --json          # JSON output
```

Logs stored at `/tmp/openclaw/openclaw-YYYY-MM-DD.log`. Verbose: `openclaw gateway --verbose`.

## Authentication

Required by default on non-loopback bindings.

- Token: `gateway.auth.token` or `OPENCLAW_GATEWAY_TOKEN`
- Password: `gateway.auth.password` or `OPENCLAW_GATEWAY_PASSWORD`
- Generate: `openclaw doctor --generate-gateway-token`
- Rate limiting: 10 failures in 60s triggers 5-minute lockout (configurable)

## Remote Access

- **Tailscale Serve:** `gateway.tailscale.mode: "serve"` (tailnet-only HTTPS)
- **Tailscale Funnel:** `gateway.tailscale.mode: "funnel"` (public HTTPS, requires password auth)
- **SSH tunnel:** `ssh -N -L 18789:127.0.0.1:18789 user@host`

## Channel Health Monitoring

- `channelHealthCheckMinutes`: 5 (default)
- `channelStaleEventThresholdMinutes`: 30 (default)
- `channelMaxRestartsPerHour`: 10 (default)
- Auto-restarts unhealthy channels

## Multi-Gateway

For true isolation, run separate gateway processes. Each needs unique port, config path, state dir.

```bash
openclaw --profile work gateway --port 18790
```

The `--profile` flag isolates state under `~/.openclaw-<name>`.
