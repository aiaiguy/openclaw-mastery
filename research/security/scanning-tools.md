# OpenClaw Security Scanning Tools

> Last updated: 2026-03-20

## 1. openclaw-security-monitor (59-Point Scan)

**Source:** github.com/adibirzu/openclaw-security-monitor

Defensive security tool for self-hosted OpenClaw. Proactive scanning, threat detection, real-time monitoring.

### Install

```bash
git clone https://github.com/adibirzu/openclaw-security-monitor.git \
  ~/.openclaw/workspace/skills/openclaw-security-monitor

cd ~/.openclaw/workspace/skills/openclaw-security-monitor
chmod +x scripts/*.sh scripts/remediate/*.sh
```

### Commands

```bash
./scripts/scan.sh                    # Run 59-point scan (read-only)
./scripts/remediate.sh --dry-run     # Preview fixes
./scripts/clawhub-scan.sh            # Scan installed skills against IOC database
./scripts/update-ioc.sh              # Update threat intelligence feeds
node dashboard/server.js             # Web dashboard on localhost:18800
```

### What the 59-Point Scan Checks

**Critical (16 checks):**
1. C2 infrastructure connections (known C2 IPs)
2. AMOS/NovaStealer/AuthTool infostealer patterns
3. Reverse shells (bash/python/perl/ruby/php/lua)
4. Credential exfiltration (webhook.site, pipedream, ngrok, burpcollaborator)
5. Memory poisoning (prompt injection in SOUL.md, MEMORY.md, IDENTITY.md)
6. Gateway config issues (auth disabled, LAN exposure, version checks)
7. WebSocket security (CVE-2026-25253 origin validation)
8. Malicious ClawHub publishers (blacklist matching)
9. Tool policy audit (elevated tools with wildcard access)
10. Plugin/extension audit (extensions with exec patterns)
11. Reverse proxy bypass
12. Exec-approvals audit (unsafe remote execution approvals)
13. Docker security (root containers, socket mounts, privileged mode)
14. VS Code trojans (fake ClawdBot/OpenClaw extensions)
15. MCP server security (unrestricted servers, prompt injection)
16. safeBins bypass (CVE-2026-28363, CVSS 9.9)

**Warning (43 checks):** File permissions, skill integrity hashes, SKILL.md injection, crypto wallet targeting, curl-pipe attacks, base64 obfuscation, binary downloads, environment leakage, DM policy audit, sandbox config, mDNS exposure, session permissions, persistence mechanisms, log redaction, Node.js CVEs, plaintext credentials, internet exposure, SSRF, PATH hijacking, and more.

### IOC Database

- `ioc/c2-ips.txt` — Known command and control IPs
- `ioc/malicious-domains.txt` — Known malicious domains
- `ioc/file-hashes.txt` — Known malicious file hashes
- `ioc/malicious-publishers.txt` — Known malicious ClawHub publishers
- `ioc/malicious-skill-patterns.txt` — Known malicious skill patterns

Minimum safe OpenClaw version per the tool: **v2026.2.26**.

---

## 2. Cisco Skill Scanner

**Source:** github.com/cisco-ai-defense/skill-scanner

Open-source security analysis from Cisco. Detects prompt injection, data exfiltration, and malicious code patterns in AI agent skills. Apache 2.0 licence.

### Install

```bash
pip install cisco-ai-skill-scanner

# With cloud provider support
pip install cisco-ai-skill-scanner[bedrock]   # AWS
pip install cisco-ai-skill-scanner[vertex]    # Google
pip install cisco-ai-skill-scanner[azure]     # Azure OpenAI
```

Requires Python 3.10+.

### Usage

```bash
# Scan a single skill
skill-scanner scan /path/to/skill

# With behavioural + LLM analysis
skill-scanner scan /path/to/skill --use-behavioral --use-llm

# Scan all skills recursively
skill-scanner scan-all /path/to/skills --recursive --use-behavioral

# CI/CD with failure threshold
skill-scanner scan-all ./skills --fail-on-severity high --format sarif

# Generate policy template
skill-scanner generate-policy -o policy.yaml
```

### 8 Detection Engines

1. **Static analysis** — YAML and YARA pattern matching
2. **Bytecode inspection** — Python .pyc integrity checks
3. **Pipeline analysis** — Command taint analysis for shell pipelines
4. **Behavioural analyser** — AST-based dataflow analysis of Python files
5. **LLM analyser** — Semantic analysis using language models (supports consensus runs)
6. **Meta-analyser** — False positive filtering and result prioritisation
7. **VirusTotal integration** — Hash-based malware detection
8. **AI Defense Cloud Service** — Cisco's cloud-based analysis

### Output Formats

`summary`, `json`, `markdown`, `table`, `sarif` (GitHub Code Scanning), `html` (interactive with attack correlation)

### Limitation

From Cisco's docs: "A scan that returns no findings does not guarantee that a skill is secure, benign, or free of vulnerabilities." Manual review remains essential.

---

## 3. OpenClaw Built-in Security Tools

```bash
openclaw security audit              # Standard security check
openclaw security audit --deep       # Comprehensive assessment
openclaw security audit --fix        # Auto-fix permissions and tighten defaults
openclaw doctor                      # General health check
openclaw doctor --deep               # Thorough audit
openclaw secrets audit --check       # Find plaintext credentials
```

---

## Recommended Scanning Workflow

For workshop participants and production deployments:

1. **Before first use:** Run `openclaw security audit --deep` and `openclaw secrets audit --check`
2. **Before installing any skill:** Run Cisco Skill Scanner (`skill-scanner scan /path/to/skill --use-behavioral`)
3. **After installation:** Run openclaw-security-monitor 59-point scan
4. **Weekly:** Run full 59-point scan and update IOC database
5. **On any change:** Re-run `openclaw security audit --deep`

This is the tiered approach:
- **Tier 1 (Weekly, minimum):** Built-in security audit + secrets audit
- **Tier 2 (Before any skill install):** Cisco Skill Scanner
- **Tier 3 (Weekly-Nightly, for production):** Full 59-point scan with IOC updates
