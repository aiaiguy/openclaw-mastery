# OpenClaw Docker Sandbox Setup

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai/install/docker, docs.openclaw.ai/gateway/sandboxing, docs.openclaw.ai/gateway/security

## Workshop-Ready Setup (Step by Step)

### Prerequisites
- Docker Desktop or Docker Engine with Compose v2
- Minimum 2 GB RAM (builds fail with exit 137 on 1 GB)

### Installation

```bash
# Clone the repo
git clone https://github.com/openclaw/openclaw
cd openclaw

# Option A: Automated (recommended for workshops)
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./scripts/docker/setup.sh

# Option B: Manual
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway

# Build sandbox image
scripts/sandbox-setup.sh
```

### Verify

```bash
curl -fsS http://127.0.0.1:18789/healthz    # Liveness
curl -fsS http://127.0.0.1:18789/readyz      # Readiness
docker compose run --rm openclaw-cli dashboard --no-open
```

---

## Hardened Configuration

```json5
{
  gateway: {
    bind: "loopback",
    auth: { mode: "token" }
  },

  agents: {
    defaults: {
      sandbox: {
        mode: "all",              // Sandbox every execution
        scope: "session",         // Isolated container per session
        workspaceAccess: "ro",    // Read-only workspace mount
        docker: {
          network: "none",        // Deny-by-default networking
          readOnlyRoot: true,     // Immutable filesystem
          capDrop: ["ALL"],       // Drop all Linux capabilities
          user: "1000:1000",      // Non-root execution
          pidsLimit: 256,         // Limit process count
          tmpfs: ["/tmp", "/var/tmp"]
        }
      }
    }
  },

  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false }
  }
}
```

### File Permissions

```bash
chmod 700 ~/.openclaw/
chmod 600 ~/.openclaw/openclaw.json
```

---

## Sandbox Scope Options

| Scope | Container Lifecycle | When to Use |
|-------|-------------------|-------------|
| `"session"` (default) | One container per session | Maximum isolation. Workshops, demos, untrusted content. |
| `"agent"` | One container per agent, shared across sessions | Balance of isolation and efficiency. Trusted internal agent. |
| `"shared"` | One container for all agents | Maximum efficiency, minimum isolation. Single-user only. |

**Workshop recommendation:** Use `"session"`. Each participant fully isolated. Compromise in one session cannot affect another.

---

## Workspace Mount Options

| Mode | Effect |
|------|--------|
| `"none"` (default) | Isolated sandbox workspace. No host access. |
| `"ro"` | Agent workspace mounted read-only at `/agent` |
| `"rw"` | Agent workspace mounted read-write at `/workspace` |

Custom mounts for specific directories:
```json5
{
  sandbox: {
    workspaceAccess: "none",
    docker: {
      binds: [
        "/home/user/datasets:/data:ro",
        "/home/user/output:/output:rw"
      ]
    }
  }
}
```

---

## Network: Deny-by-Default

Docker sandbox containers run with **no network by default** (`network: "none"`).

- Set to `"bridge"` only if outbound is explicitly required
- `"host"` and `"container:<id>"` are blocked by default
- Dangerous path blocking is automatic: docker.sock, /etc, /proc, /sys, /dev

Namespace join protection requires explicit break-glass override:
```json5
agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true
```

---

## Credential Injection (SecretRef System)

The gateway resolves credentials eagerly at startup into an in-memory snapshot. Agents never see plaintext keys.

**How it works:**
- Credentials resolved at activation time, not request time
- Agent config files store references, not values
- Gateway uses credentials for outbound API calls. Agent execution environment gets nothing.
- Reload uses atomic swap: full success or keep last-known-good

**Three sources:**
```json5
// Environment variable
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }

// File-based (supports JSON pointer)
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }

// External (Vault, 1Password, sops)
{ source: "exec", provider: "vault", id: "openai_key" }
```

**Vault integration example:**
```json5
{
  secrets: {
    providers: {
      vault: {
        source: "exec",
        command: "/usr/local/bin/vault",
        args: ["kv", "get", "-field=KEY", "secret/path"],
        passEnv: ["VAULT_TOKEN"]
      }
    }
  }
}
```

**Audit:**
```bash
openclaw secrets audit --check     # Find plaintext values and unresolved refs
openclaw secrets configure         # Interactive setup
openclaw secrets apply             # Apply saved plans
```

---

## Tailscale Serve vs Funnel

### Serve (Recommended)

| Aspect | Detail |
|--------|--------|
| Access | Only devices on your tailnet |
| Public exposure | None |
| Identity headers | Yes. Auto-added, spoofing prevented. |
| ACL integration | Full |
| Risk level | Low |

### Funnel (Avoid for OpenClaw)

| Aspect | Detail |
|--------|--------|
| Access | Anyone on the public internet |
| Public exposure | Full. Unique public URL created. |
| Identity headers | No. Cannot identify who connects. |
| Ports | Limited to 443, 8443, 10000 |
| Risk level | High for OpenClaw |

**OpenClaw docs explicitly flag Tailscale Funnel as a security issue in their audit tool.** Never use Funnel for OpenClaw unless time-limited with token auth enabled.

---

## Workshop Quick Reference Commands

```bash
# Setup
git clone https://github.com/openclaw/openclaw && cd openclaw
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./scripts/docker/setup.sh
scripts/sandbox-setup.sh

# Verify
curl -fsS http://127.0.0.1:18789/healthz

# Security
openclaw security audit --deep
openclaw secrets audit --check
chmod 700 ~/.openclaw/ && chmod 600 ~/.openclaw/openclaw.json

# Management
docker ps
docker compose run --rm openclaw-cli dashboard --no-open
```
