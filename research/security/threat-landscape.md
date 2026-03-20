# OpenClaw Security Threat Landscape

> Last updated: 2026-03-20

## The Headlines

- **Gartner**: "Insecure by default." Security risks "unacceptable."
- **Cisco**: "A security nightmare for casual users."
- **OpenClaw's own docs**: "There is no 'perfectly secure' setup."
- **CrowdStrike**, **Microsoft**, **AuthMind** have all published security advisories.

## Primary Threats

### 1. Prompt Injection
The biggest risk. Malicious prompts can be embedded in:
- Emails the agent reads
- Documents it processes
- Slack/messaging content
- File metadata
- Web pages it browses

### 2. Data Exfiltration
Skills can execute silent network calls to external servers. Demonstrated in the wild.

### 3. Malicious Skills (Supply Chain)
- 230+ malicious skills uploaded to ClawHub since Jan 2026.
- Skills can contain hidden instructions, data harvesting, or backdoors.
- No mandatory security review before publishing.

### 4. Credential Exposure
Agent has access to whatever you give it. If it has your API keys, email, or calendar, a prompt injection can weaponise those.

## Key Security Resources

| Source | Focus |
|---|---|
| Microsoft Security Blog | Identity, isolation, runtime risk |
| SlowMist Security Guide | Comprehensive hardening practices |
| CrowdStrike Advisory | What security teams need to know |
| Nvidia NemoClaw | Sandboxing + Nemotron models for safer deployment |
| AuthMind | Supply chain risks from malicious skills |

## TODO: Research Gaps

- [ ] Full hardening checklist (from SlowMist guide)
- [ ] Docker/VM isolation setup guide
- [ ] Skill vetting process and safe skill sourcing
- [ ] Network segmentation recommendations
- [ ] Credential management best practices
- [ ] Monitoring and alerting for anomalous agent behaviour
- [ ] Microsoft's specific recommendations (detailed)
- [ ] Nvidia NemoClaw as a security layer
