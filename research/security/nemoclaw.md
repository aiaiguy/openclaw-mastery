# NemoClaw: NVIDIA's Enterprise Security Wrapper for OpenClaw

> Last updated: 2026-03-21 | Status: Early Preview / Alpha (released 16 March 2026 at GTC 2026)
> Sources: NVIDIA official, TechCrunch, VentureBeat, Particula, CrowdStrike, community analysis

---

## What Is NemoClaw?

NVIDIA's open-source enterprise security and privacy stack for OpenClaw. Not a fork or replacement. A wrapper that adds enterprise-grade controls on top of OpenClaw without modifying agent code.

- **Released:** 16 March 2026 at GTC 2026
- **Licence:** Apache 2.0 (free, open source)
- **GitHub:** github.com/NVIDIA/NemoClaw (4,600 stars, 20 contributors in first week)
- Jensen Huang called OpenClaw "the next ChatGPT" and "as big a deal as Linux"
- NVIDIA worked directly with OpenClaw creator Peter Steinberger

---

## Three Core Components

### 1. OpenShell Runtime (Kernel-Level Sandbox)

Open-source runtime (github.com/NVIDIA/OpenShell) that sandboxes agents at the process level using Linux kernel security primitives.

| Layer | Technology | What It Controls |
|---|---|---|
| **Filesystem** | Landlock LSM | Agents restricted to `/sandbox` and `/tmp`. System paths read-only. Locked at sandbox creation, cannot be modified at runtime. |
| **System calls** | seccomp | Blocks privilege escalation. Agent-optimised profile (not Docker's general-purpose default). |
| **Network** | Network namespaces | Blocks ALL outbound except explicitly allowed hosts. Hot-reloadable. |
| **Process** | Out-of-process enforcement | Policy engine in separate process. Agent cannot access, modify, or terminate it. |

**Deny-by-default model:** Agent can reach the OpenShell gateway and configured inference endpoint. Everything else blocked. Unknown hosts intercepted and surfaced in a TUI for human approval.

### How It Differs from Docker Sandboxing

This is the critical distinction.

**Docker** operates at the infrastructure level. A containerised OpenClaw agent still has unrestricted network access, unrestricted filesystem freedom, and can call any inference endpoint *inside* its container. Docker isolates processes from each other but does not constrain agent behaviour.

**OpenShell** operates at the application level *inside* the container. It adds four policy layers that constrain the agent's actual behaviour. Even running inside Docker, OpenShell adds filesystem, network, process, and inference controls Docker alone does not provide.

**Key design principle:** Out-of-process enforcement. If the agent is fully compromised, it cannot disable the sandbox. OpenClaw's native security model enforces permissions within the agent framework itself. If a malicious skill compromises the agent process, it can modify its own permission checks. OpenShell eliminates this.

### 2. Policy Engine (YAML-Based Governance)

Policies live in `openclaw-sandbox.yaml`.

**Filesystem policies (locked at creation):**
```yaml
filesystem_policy:
  read_only:
    - /usr
    - /lib
    - /etc
  read_write:
    - /sandbox
    - /tmp
    - /dev/null
```

**Network policies (hot-reloadable):**
```yaml
network_policies:
  nvidia:
    endpoints:
      - { host: integrate.api.nvidia.com, port: 443 }
    binaries:
      - { path: /usr/bin/python3 }
  slack:
    name: slack
    endpoints:
      - host: api.slack.com
        port: 443
        protocol: rest
        enforcement: enforce
        tls: terminate
        rules:
          - allow: { method: GET, path: "/**" }
          - allow: { method: POST, path: "/**" }
```

**Presets shipped:** Slack, Jira, Discord, npm, PyPI, Docker, Telegram, HuggingFace, Outlook.

**vs OpenClaw's native tool allow/deny:** OpenClaw's permissions are in-process (agent can theoretically override them). NemoClaw's are out-of-process at the kernel level. Agent cannot override, disable, or modify them.

### 3. Privacy Router

Classifies queries by data sensitivity and routes accordingly.

- **High-sensitivity** (PII, proprietary data, NDAs): Routed to local Nemotron models. Data never leaves device/VPC.
- **Low-sensitivity** (general reasoning, public info): Can route to cloud frontier models for performance.

**PII handling:** Redaction, tokenisation, policy context attachment. Uses differential privacy technology from Gretel acquisition. [Unverified: specific technical details of Gretel integration not published as of 21 March 2026.]

**Not locked to NVIDIA models.** Works with OpenAI, Anthropic, and other providers. Privacy router governs what data reaches which endpoint.

---

## Nemotron Models

| Model | Parameters | Notes |
|---|---|---|
| **Nemotron 3 Nano 4B** | 4 billion | Lightweight, runs on consumer RTX hardware |
| **Nemotron 3 Super 120B** | 120 billion | Top open model on PinchBench (85.6%) |
| **Nemotron 3 Ultra** | Not specified | Frontier-level, 5x throughput on Blackwell |

PinchBench is a new benchmark specifically for measuring LLM performance with OpenClaw.

### Hardware Requirements

| Platform | Memory | Notes |
|---|---|---|
| GeForce RTX PCs/laptops | Varies by GPU | Entry point, runs Nano 4B |
| RTX PRO 6000 Blackwell | 96 GB GPU | Workstation-grade |
| DGX Spark | 128 GB unified | Supports >120B parameter models. ~$3,999. |
| DGX Station | 748 GB coherent | GB300 Grace Blackwell Ultra. ~$50,000+ [Unverified] |

### Nemotron Coalition

Eight AI labs collaborating on open frontier models: Black Forest Labs, Cursor, LangChain, Mistral AI, Perplexity, Reflection AI, Sarvam, Thinking Machines Lab. First model will underpin upcoming Nemotron 4 family.

---

## Installation

### One-Command Install

```bash
curl -fsSL https://www.nvidia.com/nemoclaw.sh | bash
```

### Onboarding Wizard (7 steps, 10-15 minutes)

```bash
nemoclaw onboard
```

1. Downloads blueprint artifact
2. Checks version compatibility, verifies digest
3. Creates OpenShell sandbox (isolated container)
4. Configures inference provider
5. Applies initial security policies
6. Sets up local model inference (if hardware supports it)
7. Runs validation checks

### Post-Install

```bash
source ~/.bashrc  # or source ~/.zshrc
nemoclaw connect   # Connect to sandbox
nemoclaw chat      # Launch interactive chat
```

### Platform Support

- **Linux:** Native support. Full kernel-level isolation (Landlock, seccomp, network namespaces).
- **macOS/Apple Silicon:** Via Docker Desktop, but with significant gaps (see Limitations).
- **Windows:** Not directly specified. [Inference: likely via WSL2/Docker.]
- **Cloud:** DigitalOcean 1-click deployment available. NVIDIA Brev support documented.

---

## Enterprise Partners (17 Confirmed)

| Partner | Integration |
|---|---|
| **Salesforce** | NVIDIA Agent Toolkit + Nemotron into Agentforce |
| **SAP** | Agent Toolkit with NeMo through Joule Studio |
| **CrowdStrike** | Falcon platform protection embedded in OpenShell (AIDR, Endpoint, Cloud, Identity Security) |
| **Adobe** | Creative workflow automation |
| **ServiceNow** | Launch partner |
| **Siemens** | Launch partner |
| **Atlassian** | Launch partner |
| **Palantir** | Launch partner (defence) |
| **Red Hat** | Launch partner |
| **Cisco** | Launch partner |
| **Box, Cohesity, Cadence, Synopsys, IQVIA, Dassault Systemes, Amdocs** | Launch partners |

CrowdStrike integration is the most technically detailed. Falcon AIDR secures every prompt, response, and agent action.

---

## Limitations (Honest Assessment)

### Alpha Quality
NVIDIA explicitly warns: "Expect rough edges." APIs and interfaces may change without notice. Not production-ready by any enterprise definition.

### macOS/Apple Silicon Is Broken
- Local inference via Ollama broken (DNS bug in sandbox setup)
- Docker Model Runner can't reach through sandbox network namespace
- Telegram and Discord integrations fail with 403 errors on M4 Macs
- No `setup-apple` equivalent to `setup-spark`

### Linux-First
Kernel-level isolation (Landlock, seccomp, network namespaces) is Linux-native. macOS depends on Docker Desktop as translation layer.

### No Compliance Certifications
Targeting SOC 2 and GDPR, but nothing certified today.

### No Performance Benchmarks
NemoClaw-wrapped vs bare OpenClaw comparisons not published.

### Behavioural Governance Is Missing
**This is the sharpest criticism.** NemoClaw prevents an agent from accessing files it shouldn't. It does NOT prevent an agent from making bad decisions with files it CAN access. An agent with legitimate codebase access can find a bug, write a plausible but untested fix, commit it, push it, and report success. NemoClaw's sandbox would allow every step because the agent had permission. The damage comes from "slow, confident degradation by an agent that has every permission it needs and no judgement about how to use them."

### NVIDIA Hardware Dependency
Local inference runs best (or only) on NVIDIA GPUs. Full local-inference story requires NVIDIA hardware. [Inference: this is NVIDIA's GPU moat strategy playing out through software.]

### Small Community
4,600 stars and 20 contributors vs OpenClaw's 250K+ stars.

### Enterprise Tier Pricing Unknown
NVIDIA will offer managed infrastructure, compliance tooling, and support SLAs. Pricing not announced. [Unverified: expected usage-based tied to compute consumption.]

---

## Comparison Matrix

| Dimension | Bare OpenClaw | Docker-Hardened | NemoClaw | Microsoft Copilot |
|---|---|---|---|---|
| **Security model** | In-process. Agent can override. | Container isolation. Agent free inside. | Kernel-level, out-of-process. Agent cannot override. | Cloud-managed. |
| **Filesystem** | AGENTS.md allow/deny (in-process) | Container boundaries only | Landlock LSM, deny-by-default, locked | N/A (SaaS) |
| **Network** | None by default | Container networking | Network namespaces, deny-by-default, hot-reloadable | Microsoft-managed |
| **PII handling** | None | None | Privacy router with PII stripping | Purview integration |
| **Local inference** | Possible but unmanaged | Possible but unmanaged | Managed Nemotron, privacy-aware routing | No (cloud only) |
| **Compliance** | None | None | Targeting SOC 2/GDPR (not certified) | SOC 2, ISO 27001, GDPR certified |
| **Cost** | Free + API | Free + infra + API | Free + hardware + API + enterprise tier TBD | $30/user/month |
| **Maturity** | Production-used | Depends on config | Alpha | GA, enterprise-proven |

---

## The Board Question: Should We WAIT for NemoClaw?

### Case for Waiting
- Solves the exact security problems that make boards nervous
- Out-of-process enforcement is architecturally superior
- CrowdStrike integration gives security teams familiar tooling
- Privacy router addresses data sovereignty
- 17 enterprise partners signal serious momentum

### Case for NOT Waiting
- **Alpha.** No GA timeline. No SLA. No compliance certifications.
- OpenClaw is already deployable with Docker sandboxing and careful configuration
- The market is not waiting. 6-12 months delay means falling behind.
- NemoClaw solves security, not judgement
- Hardware lock-in to NVIDIA

### Recommended Board Position

**Do not wait. Start now with hardened OpenClaw. Plan to migrate to NemoClaw when it reaches GA.**

1. Deploy OpenClaw now with Docker sandboxing and strict policies
2. Run controlled pilots with low-risk use cases
3. Track NemoClaw development, begin testing when it exits alpha
4. Plan migration for production workloads handling sensitive data once compliance-certified
5. Budget for NVIDIA hardware if local inference and data sovereignty are requirements

---

## Key Dates

| Date | Event |
|---|---|
| 16 March 2026 | NemoClaw announced at GTC, early preview released |
| 16 March 2026 | CrowdStrike Secure-by-Design Blueprint announced |
| 16 March 2026 | Nemotron Coalition announced (8 AI labs) |
| TBD | NemoClaw beta |
| TBD | NemoClaw GA |
| TBD | SOC 2 / GDPR certification |
| TBD | Enterprise tier pricing |

---

## Sources

- NVIDIA NemoClaw Press Release: nvidianews.nvidia.com/news/nvidia-announces-nemoclaw
- NVIDIA NemoClaw Product Page: nvidia.com/en-us/ai/nemoclaw/
- NVIDIA NemoClaw Developer Docs: docs.nvidia.com/nemoclaw/latest/
- GitHub NVIDIA/NemoClaw: github.com/NVIDIA/NemoClaw
- GitHub NVIDIA/OpenShell: github.com/NVIDIA/OpenShell
- NVIDIA OpenShell Technical Blog: developer.nvidia.com/blog/run-autonomous-self-evolving-agents-more-safely-with-nvidia-openshell/
- TechCrunch: techcrunch.com/2026/03/16/nvidias-version-of-openclaw-could-solve-its-biggest-problem-security/
- VentureBeat: venturebeat.com/technology/nvidias-nemoclaw-brings-privacy-and-security-controls-to-autonomous-openclaw
- Particula: particula.tech/blog/nvidia-nemoclaw-openclaw-enterprise-security
- CrowdStrike Blueprint: crowdstrike.com/en-us/press-releases/crowdstrike-nvidia-unveil-secure-by-design-ai-blueprint-for-ai-agents/
- Stormap Architecture Analysis: stormap.ai/post/inside-nemoclaw-the-architecture-sandbox-model-and-security-tradeoffs
- Katonic Docker Comparison: katonic.ai/blog/nemoclaw-docker-isolation
- Augmented Mind Criticism: augmentedmind.substack.com/p/nemoclaw-is-not-the-fix-here-is-what-is-missing
