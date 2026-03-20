# OpenClaw Security Threat Landscape

> Last updated: 2026-03-20 | Sources: Microsoft, CrowdStrike, Gartner, Cisco, CSA, SlowMist, Trend Micro, VentureBeat, and others cited below

## The Headline Assessments

- **Gartner**: "Agentic Productivity Comes With Unacceptable Cybersecurity Risk." Labelled "insecure by default."
- **Microsoft Defender**: "Not appropriate to run on a standard personal or enterprise workstation."
- **CrowdStrike**: "If employees deploy OpenClaw on corporate machines, it could be commandeered as a powerful AI backdoor agent."
- **Cisco**: Ran a live skill scan. Result: 9 security findings, 2 critical, 5 high severity. "OpenClaw fails decisively."
- **OpenClaw's own docs**: "There is no 'perfectly secure' setup."

---

## The Full Threat Model

### 1. Prompt Injection

The biggest risk. OpenClaw reads content from external sources. Anyone who controls that content can embed hidden instructions.

**Attack surfaces:**
- Email bodies and headers
- Shared documents (Google Docs, files)
- Slack/messaging content
- Web pages during browsing
- File metadata
- Link previews (Telegram/Discord turned into exfiltration pathway)

**Persistent memory attacks:** Malicious payloads fragmented across time. Injected into MEMORY.md on day one, detonated when agent state aligns later. Time-shifted prompt injection, memory poisoning, logic-bomb attacks.

### 2. Data Exfiltration

- **Semantic exfiltration**: Agent sending emails looks identical to normal activity. EDR sees HTTP 200. Payload is natural language, not malicious code.
- **Link preview exfiltration**: Tricking agent into generating URLs that encode sensitive data, auto-fetched by messaging apps.
- **Credential harvesting**: Config files, memory, and chat logs store API keys in plain text. RedLine and Lumma infostealers have added OpenClaw file paths to their must-steal lists.

### 3. Supply Chain Attacks (ClawHavoc)

- Researcher audited all 2,857 skills on ClawHub: found 341 malicious entries.
- Grew to **824+ malicious skills across 10,700+ in registry (~20% of all skills)**.
- All used same playbook: fake prerequisite installations silently deploying **Atomic macOS Stealer (AMOS)**.
- Professional documentation, innocuous names like "solana-wallet-tracker."

### 4. Lateral Movement

Multi-agent architectures pass context (session tokens, credentials, document fragments) between agents without privilege separation. IAM systems cannot detect this. Each agent operates under apparently valid identity.

### 5. EDR/DLP/IAM Bypass

Per VentureBeat: OpenClaw bypasses EDR because agent behaviour looks normal. Credentials are real, API calls sanctioned. Nothing in current defence ecosystem tracks what the agent decided to do with that access, or why.

### 6. CSA MAESTRO Threat Model (Seven Layers)

Cloud Security Alliance applied their MAESTRO framework. Threats identified at every layer:
1. Foundation Models: prompt injection, API key exposure
2. Data Operations: plaintext credentials, world-readable state dirs
3. Agent Frameworks: tool misuse, session spawning abuse
4. Deployment: gateway binding, Tailscale misconfiguration, Docker socket exposure
5. Evaluation: insufficient anomaly detection, audit logs vulnerable to tampering
6. Security: DM policy misconfiguration, group chat auth bypasses
7. Ecosystem: malicious plugins, supply chain, multi-agent collusion

---

## Known Security Incidents

### CVE-2026-25253: One-Click Remote Code Execution (CVSS 8.8)
Control UI accepted `gatewayUrl` query parameter without validation, transmitted user auth token to attacker-controlled server. Pivots through victim's browser. Patched v2026.1.29.

### ClawHavoc Supply Chain Campaign
341 initial, 824+ total malicious skills. Delivered Atomic macOS Stealer. Targeted crypto users and developers.

### ClawJacked WebSocket Hijacking
Malicious sites hijack local OpenClaw agents via WebSocket. Exploits lack of origin validation.

### 135,000+ Internet-Exposed Instances
SecurityScorecard found 135,000+ instances exposed in 82 countries. 15,000+ directly vulnerable to RCE. 93.4% of verified instances exhibited authentication bypass.

### Moltbook Database Breach
Social network for OpenClaw agents. Unsecured database exposed 35,000 emails and 1.5 million agent API tokens.

### Infostealer Targeting
RedLine and Lumma added `~/.openclaw/` to their must-steal file paths. Targeting plaintext credentials.

---

## Key Takeaways for Workshop

1. **This is not theoretical.** CVEs, supply chain campaigns, 135k exposed instances, infostealers targeting it by name.
2. **Every major security vendor agrees:** not suitable for standard workstations without hardening.
3. **The security gap is architectural:** EDR/DLP/IAM cannot see what an AI agent decides to do with valid credentials.
4. **20% of ClawHub skills were malicious.** Supply chain risk is real.
5. **If your people are using it (and they probably are), visibility first, policy second, isolation or removal third.**
