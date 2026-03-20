# Challenge Findings: What We Got Wrong, What We Missed, What Changes

> Last updated: 2026-03-20 | This is the critical "second pass" that stress-tests our initial research.

## Critical Blind Spots We Missed

### 1. Sandbox Escape Rate: 83%
An arxiv paper (2603.10387v1) found OpenClaw's average defence rate against sandbox escape attacks is only **17%**. That means 83% of attempted breakouts succeed. This dramatically undermines any "deploy in Docker and you're fine" advice. Sandboxing is a layer, not a solution.

### 2. Log Integrity Problem
A security audit on AWS Lightsail found that **logs are deletable by the agent itself**. If an agent is compromised, it can cover its tracks. This matters for compliance and forensics. The governance framework needs immutable, external logging (ship logs to a separate SIEM that the agent cannot access).

### 3. Prompt Injection Is Architectural, Not a Bug
The deeper threat beyond CVEs and malicious skills: prompt injection is an architectural property of LLM-based agents. It cannot be patched. It can only be managed. Giskard research showed crafted prompts can extract API keys and environment variables from running agents. The workshop must name this explicitly and set expectations: **this risk is managed, not eliminated.**

### 4. Founder Left for OpenAI
Peter Steinberger joined OpenAI on 14 February 2026. The original creator is no longer steering the open-source project. Governance, roadmap stability, and long-term maintenance are now open questions. Bus-factor risk for any organisation building workflows on OpenClaw.

### 5. China Restricted It
Chinese authorities (CNCERT, MIIT) have restricted state-run enterprises and banks from running OpenClaw on office devices. Signal that sovereign risk assessments are already happening. Relevant for Australian government and regulated-sector clients.

---

## Where Our Advice Was Too Aggressive

### Workshop Hands-On Deployment: CHANGE THIS
We proposed "every participant leaves with a working deployment." This is dangerous for a C-suite audience.

If non-technical board members take a deployment home and run it on a corporate laptop without hardening, we've created exactly the shadow IT problem we're trying to solve. The 83% sandbox escape rate makes this worse.

**New recommendation:** Participants leave with:
- A documented **security policy template**
- A **deployment request template** for their IT team
- A **governance framework** they can present to their board
- Live demos run on a pre-built, locked-down environment (not their machines)
- If hands-on is essential: use a VM image that **expires after 48 hours**

### The 13-Point Nightly Audit Is Unrealistic
SlowMist's framework is designed for crypto/DeFi teams with dedicated security engineers. A mid-market Australian business does not have the team to sustain nightly 13-point audits.

**New recommendation:** Tier the audit framework:
- **Tier 1 (Weekly, 5 checks):** For low-risk use cases (personal productivity, no customer data)
- **Tier 2 (Nightly, full 13):** For anything touching customer data, financial systems, or regulated data

---

## Where Our Advice Was Too Conservative

### "Insecure by Default" Is True But Misleading
OpenClaw is insecure by default in the same way Linux, Docker, and Kubernetes are insecure by default. Every powerful infrastructure tool ships this way because security requires context.

The question is not "is it insecure?" but **"can it be hardened to an acceptable level for the use case?"** For low-risk personal productivity (daily briefings, email triage, meeting prep), the answer is yes, and cheaply.

### Cost Is Not the Problem
At $6-50/month AUD for personal to small business use, OpenClaw is cheaper than most SaaS subscriptions. Framing costs without comparing to alternatives (hiring a VA at $40/hr, paying for Zapier at $75/month) loses the economic argument.

**The real cost is security overhead.** The tool is cheap. Hardening, auditing, monitoring, and incident response require skilled staff. For a company without a dedicated security team, this operational cost may exceed the tool's value for anything beyond basic personal use.

### Banning It Outright Is Counterproductive
Google and Meta banned it, but **22% of monitored organisations already have employees running it without approval.** Prohibition creates shadow IT. The governance response should be "sanctioned use with guardrails," not "no use."

---

## The Counterargument: "Why Would We Use This?"

Three-part honest response:

1. **The risk of inaction is also real.** Competitors adopting agentic AI will move faster. Jensen Huang compared OpenClaw to Linux and Kubernetes. Whether you agree or not, the category is not going away. The question is whether you adopt with governance or your employees adopt without it.

2. **The security ecosystem is hardening rapidly.** VirusTotal partnership for skill scanning (Feb 2026). NemoClaw kernel-level sandboxing (Mar 2026). AWS managed deployments. The trajectory is toward enterprise-grade security.

3. **Start with low-stakes, high-value use cases.** Daily briefings, email triage, research summarisation, meeting prep. No access to financial systems or customer PII. Prove value. Expand with controls.

---

## Regulatory and Compliance (Australian Context)

### Australian Privacy Act, December 2026 Changes
New requirements for **automated decision-making transparency** take effect December 2026. If OpenClaw makes decisions affecting individuals (triaging complaints, screening applications), organisations must:
- Disclose that automated decision-making is occurring
- Explain how it works
- The Children's Online Privacy Code also takes effect

### APP 11 (Security of Personal Information)
The amended Act requires "technical and organisational measures" mirroring GDPR language. Running OpenClaw with default security settings on a corporate network almost certainly fails this test.

### GDPR Article 22
If OpenClaw processes EU resident data, the right to human review of automated decisions applies.

### Sector-Specific
No APRA, ASIC, or AHPRA guidance on agentic AI tools exists yet. This is a gap, not a green light. Boards should assume existing data handling obligations extend to any AI agent with system access. [Inference]

---

## Insurance and Liability

**Board members are personally exposed.** Under Australian law, directors have duties of care and diligence (Corporations Act s180). Knowingly deploying a tool with known critical vulnerabilities, or failing to govern employee use, could constitute a breach of duty.

**MIT licence means no vendor liability.** OpenClaw is open source. No vendor to sue, no SLA, no support contract. The organisation bears 100% of the risk. Fundamentally different from Microsoft Copilot or Google Gemini Enterprise where vendor contracts include liability provisions.

**Cyber insurance question for every board:** "Does your existing cyber insurance policy cover incidents caused by autonomous AI agents?" Most boards have not asked this. [Speculation] Insurers may add specific questionnaire items about agentic AI in policy renewals from H2 2026.

---

## Shadow IT: The 22% Problem

The most actionable finding for C-suite. The governance response:

1. **Discover.** Network scans for OpenClaw traffic (WebSocket on port 18789, API calls to model providers)
2. **Classify.** Low-risk personal productivity vs high-risk corporate system access
3. **Sanction with guardrails.** Approved deployment path with pre-hardened configs. Make the secure path easier than the insecure path.
4. **Monitor.** Centralised, immutable logging shipped to SIEM
5. **Educate.** The workshop is this step.

---

## The 6-Month Outlook

**What will change:**
- NemoClaw will ship. Docker manual hardening becomes obsolete for enterprise.
- ClawHub will get supply chain security (VirusTotal, signed skills, reputation scoring). [Inference]
- Managed offerings on AWS/Azure/GCP will shift security burden to cloud providers. [Inference]
- Australian regulatory guidance from OAIC/APRA will likely arrive by late 2026. [Speculation]
- Cyber insurers will add agentic AI to policy questionnaires from H2 2026. [Speculation]

**What advice given today might be wrong in 6 months:**
1. Specific hardening configs will change with each version. Teach principles, not values.
2. "NemoClaw is alpha" may become "NemoClaw is the only responsible enterprise choice."
3. Token costs will halve as model prices continue to fall. [Inference]
4. The 22% shadow IT figure will grow. Could be 40%+ by September. [Speculation]

---

## Summary: What Changes in the Workshop

1. **Do not send C-suite home with live deployments.** Governance framework + policy templates instead.
2. **Add prompt injection as unfixable architectural risk.** Set expectations honestly.
3. **Add insurance and liability as a dedicated section.** Boards care about this more than CVEs.
4. **Name the December 2026 Australian privacy deadlines.** Creates urgency.
5. **Tier the audit framework.** 13-point nightly is unsustainable. Offer realistic alternatives.
6. **Address shadow IT head-on.** 22% already using it. Governance beats prohibition.
7. **Frame cost honestly.** Tool is cheap. Security overhead is not.
8. **Build around principles, not config values.** Technical details have a 6-month shelf life.
9. **Add immutable external logging.** Agent can delete its own logs.
10. **Acknowledge bus-factor risk.** Founder left. Project governance is in transition.
