# OpenClaw Security Hardening Guide

> Last updated: 2026-03-20 | Sources: SlowMist, Microsoft, CrowdStrike, Docker, Cisco, OpenClaw official docs

## Microsoft's Recommended Deployment Model

The strongest guidance from the most credible source:

1. Deploy only in a **fully isolated environment** (dedicated VM or separate physical machine)
2. Use **dedicated, non-privileged credentials**
3. Access only **non-sensitive data**
4. **Continuous monitoring** as part of operating model
5. **Assume compromise**. A rebuild plan must be part of operating model.

---

## SlowMist Three-Layer Defence Matrix

The most comprehensive hardening guide available. Shifts from traditional host-based defence to "Agentic Zero-Trust Architecture."

### Layer 1: Pre-Action (Prevention and Vetting)

**Red Line commands (absolute prohibitions, require human confirmation):**
- Destructive operations: `rm -rf /`, filesystem formatting, disk wiping
- Credential tampering: modifying auth fields, SSH settings
- Data exfiltration: curl/wget/nc transmitting tokens, keys, mnemonics
- Reverse shells
- Persistence mechanisms: unauthorised system tasks, user account mods
- Code injection: base64-decoded blind execution
- Permission tampering: unauthorised chmod/chown on core files

**Yellow Line commands (permitted with documented logging):**
- Any `sudo` operation
- Package installations, Docker run, firewall changes, service management
- `openclaw cron` modifications

**Skill installation audit:**
1. List all files: `clawhub inspect <slug> --files`
2. Full-text regex scanning for hidden instructions
3. Cross-reference against Red Line patterns
4. Await human confirmation before deployment

### Layer 2: In-Action (Runtime Protection)

- Permission narrowing: `chmod 600` on `openclaw.json` and `paired.json`
- Hash baseline: SHA256 checksums of config files, verified during nightly audits
- Business risk control: pre-flight checks before irreversible operations
- "Signature Isolation": agents construct unsigned transaction data only. Humans complete signatures via independent wallets.
- Audit script locked with kernel-level read-only lock (`chattr +i`)

### Layer 3: Post-Action (Detection and Recovery)

Nightly automated audit covering 13 core metrics (daily at 03:00):
1. Platform security audit (`openclaw security audit --deep`)
2. Network/process anomalies (listening ports, suspicious connections)
3. Sensitive directory modifications within 24 hours
4. System-level scheduled tasks and cron jobs
5. OpenClaw internal cron job inventory
6. SSH/login security (failed attempts, suspicious access)
7. Critical file integrity (hash validation, permission verification)
8. Yellow Line operation cross-validation against system logs
9. Disk usage thresholds (alert at >85%)
10. Gateway environment variables (sanitised enumeration vs whitelist)
11. Plaintext credential/private key scan (DLP) with critical alert triggering
12. Skill/MCP integrity via hash manifest diffing
13. Disaster recovery backup sync to private repository

### Honest Limitations

SlowMist openly acknowledges:
- "LLM cognitive layer highly susceptible to carefully crafted complex documents. There is no absolute security."
- Same-UID privilege problem: malicious code executes with user privileges
- Non-realtime detection: hash baseline audited nightly only (~24h latency)
- "Human common sense and secondary confirmation are the ultimate defence"

---

## Docker/VM Isolation

### Docker Sandbox Architecture

Docker Sandboxes run agents in isolated micro VMs with deny-by-default network proxy.

**Key hardening flags:**
- `--read-only` (read-only root filesystem)
- `--cap-drop=ALL` (drop all Linux capabilities)
- `--security-opt=no-new-privileges`
- Run as non-root user inside container
- Only mount specific directories agent needs

**Network proxy credential injection:** Sandbox proxy automatically injects API keys at network layer. Agent never sees raw keys and cannot leak them.

### Isolation Trade-offs

| Approach | Isolation | Convenience | Resource Cost |
|----------|-----------|-------------|---------------|
| Native install | None | Highest | Lowest |
| Docker container | Good | Medium | Low |
| Docker Sandbox (micro VM) | Strong | Medium | Medium |
| Dedicated VM | Complete | Low | High |
| Separate physical machine | Air-gapped | Lowest | Highest |

---

## Network Segmentation

- **Gateway binding:** Loopback only (default). Non-loopback requires token/password + firewall.
- **VPN-only access:** Tailscale or WireGuard. No gateway port publicly reachable.
- **Dedicated VLAN:** Separate network segment for AI agent hardware.
- **Proxy routing:** HTTP_PROXY, HTTPS_PROXY, NO_PROXY to route through monitored proxies.
- **Docker isolated network:** Agent can only reach specified external services.
- **DNS-level controls:** Block/monitor requests to openclaw.ai and clawhub domains to detect shadow IT.
- **Docker caveat:** Docker's network routing can bypass host firewalls. Requires explicit Docker network rules.

---

## Skill Vetting Process

### Safe Sourcing

1. Only install from verified, known authors
2. Read all skill source code before installation
3. Use Cisco's open-source Skill Scanner tool
4. Prefer skills with community audit history
5. Never install skills requiring external installation scripts
6. Monitor for hash changes in installed skills (nightly audit metric #12)

### Automated Scanning

```bash
clawhub inspect <slug> --files    # List all files
```

Scan for: `exec` functions, environment harvesting, network calls to unknown hosts, base64 encoding, curl/wget to external URLs.

---

## Credential Management

### The Problem

OpenClaw stores config, memory, and chat logs with API keys in plain text under `~/.openclaw/`. Infostealers target these paths.

### SecretRef System

```json5
// Best: exec provider (e.g., 1Password)
{ source: "exec", provider: "default", id: "op read op://vault/item/field" }

// Good: environment variable
{ source: "env", provider: "default", id: "ANTHROPIC_API_KEY" }

// Acceptable: file reference
{ source: "file", provider: "default", id: "/path/to/secret" }

// Bad: plain text in config
"sk-abc123"
```

### Credential Classification

| Class | Examples | Rotation Cadence |
|-------|----------|-----------------|
| Model provider keys | Anthropic, OpenAI, Google | Monthly |
| Messaging tokens | Telegram, Slack, Discord bots | Quarterly |
| Gateway auth | Token/password | Monthly |
| Tunnel/cloud | Tailscale, AWS | Quarterly |
| Deploy tokens | GitHub, npm | On change |

### Docker Credential Injection

Docker Sandboxes inject API keys at network proxy layer. Agent never sees raw key. Cannot leak it.

### CLI

```bash
openclaw secrets reload          # Re-resolve refs
openclaw secrets configure       # Interactive setup
openclaw secrets apply           # Apply from plan
openclaw security audit --fix    # Auto-fix permissions
```

---

## Monitoring and Alerting

### Built-in

- `openclaw security audit --deep` — comprehensive assessment
- `openclaw doctor` — configuration validation

### What to Monitor

- Unusual tool usage patterns (frequency, timing, type)
- Network connections to unknown hosts
- File access outside expected directories
- Authentication failures
- Config file modifications
- Credential access patterns

### Third-Party

[openclaw-security-monitor](https://github.com/adibirzu/openclaw-security-monitor): 59-point scan covering C2 infrastructure, stealers, reverse shells, credential exfiltration, memory poisoning, supply chain attacks.

### The Fundamental Gap

Current defence ecosystem (EDR, DLP, IAM) cannot track what an AI agent decides to do with valid credentials. This is the open problem.

---

## Nvidia NemoClaw

Announced GTC 2026, 16 March 2026. Installs onto OpenClaw in a single command.

### Three Components

1. **Sandbox (OpenShell Runtime):** Kernel-level, deny-by-default filesystem access
2. **Policy Engine:** Out-of-process (compromised agent cannot override). YAML-based policies.
3. **Privacy Router:** Strips PII before external sends. Local Nemotron models handle sensitive data.

### Enterprise Partners

Adobe, Salesforce, SAP, CrowdStrike, Dell.

### Production Readiness

**Early-stage alpha.** No independent security audits. Not battle-tested. Reference stack, not production-hardened. Watch this space.

---

## Governance Framework (Pre-Deployment)

### Policies Required Before Deployment

**Acceptable Use:**
- Which roles may deploy/use OpenClaw
- Which data classifications are permitted
- Which tools/skills may be enabled
- Approval workflow for new skill installations

**Data Classification (Tri-Colour):**
- Green: automated execution permitted
- Amber: human confirmation required
- Red: prohibited, hard-blocked

**Identity and Access:**
- Dedicated service accounts (never personal credentials)
- Least-privilege token design
- Credential rotation schedules
- No shared credentials between environments

**Incident Response:**
- Playbook for compromised instance (assume compromise, rebuild)
- Credential rotation triggers
- Forensic preservation of memory files and audit logs
- Communication plan for data exfiltration incidents

**Regulatory:**
- CISA: any AI with autonomous execution requires same governance as privileged access tools
- Treat OpenClaw deployments as privileged access, not casual tooling

---

## CrowdStrike's Specific Recommendations

### Discovery (First Priority)

- DNS monitoring: track requests to openclaw.ai
- Package inventory: detect npm global installations
- External exposure scanning: identify internet-exposed instances
- Process visibility: monitor OpenClaw process trees

### For Permitted Deployment

- Bind gateway exclusively to localhost
- Require strong auth tokens on all connections
- Disable high-risk tools by default (shell, browser, web fetch/search)

### Removal Workflow (If Banning)

1. Stop services and processes
2. Uninstall npm/Homebrew packages
3. Delete installation directories and PATH binaries
4. Purge service registrations and scheduled tasks
5. Remove config directories (.openclaw, .clawdbot, .clawhub)
6. Clean firewall rules

---

## Workshop Security Module Summary

The business message for C-suite:

1. **OpenClaw is not enterprise software.** No admin panel, no SSO, no compliance certifications.
2. **The threat is real and documented.** CVEs, supply chain attacks, 135k exposed instances, infostealers.
3. **Every major security vendor agrees** it should not run on standard workstations without hardening.
4. **If your people are using it (and they probably are):** visibility first, policy second, isolation or removal third.
5. **NemoClaw is the emerging answer** for capability with guardrails. Early alpha.
6. **Governance must precede deployment.** Data classification, acceptable use, credential management, incident response.
