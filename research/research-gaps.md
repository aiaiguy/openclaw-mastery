# Research Gaps and Open Questions

> Last updated: 2026-03-20

## Verified Gaps (Confirmed Not in Official Docs)

- [ ] No native MCP (Model Context Protocol) support. Uses own plugin model.
- [ ] No native email channel. Gmail Pub/Sub + skills only.
- [ ] No native calendar integration. Skills + Composio bridge.
- [ ] No formal trust-level system for skills (e.g., verified/unverified tiers)
- [ ] No automated malicious skill scanning at ClawHub registry level
- [ ] No dedicated heartbeat health-check endpoint or dashboard
- [ ] No built-in escalation framework (encode in HEARTBEAT.md instead)
- [ ] No curated "best skills" list (ClawHub too nascent)
- [ ] No dedicated rollback command for version downgrades

## Needs Deeper Investigation

- [ ] Full Docker Sandbox setup walkthrough (step-by-step for workshop)
- [ ] Tailscale Serve/Funnel security implications in detail
- [ ] OpenShell Runtime (NemoClaw) hands-on testing when available
- [ ] Composio integration setup and configuration
- [ ] n8n webhook credential isolation pattern (implementation details)
- [ ] SlowMist nightly audit script (actual implementation)
- [ ] Cisco Skill Scanner tool (hands-on evaluation)
- [ ] openclaw-security-monitor (59-point scan, hands-on evaluation)
- [ ] Real token cost data for different model + heartbeat configurations
- [ ] WhatsApp Baileys stability over time (how often does it break?)
- [ ] Identity linking across platforms (practical implementation)
- [ ] Persistent cron session patterns (implementation examples)

## Needs Testing on Justin's Setup

- [ ] Current openclaw.json configuration audit
- [ ] Current agent definitions review
- [ ] Current skill inventory and vetting
- [ ] Current heartbeat configuration and cost
- [ ] Security audit (`openclaw security audit --deep`)
- [ ] Doctor check (`openclaw doctor --deep`)
- [ ] Channel health check
- [ ] Memory architecture review
- [ ] Credential storage audit

## Workshop Content Gaps

- [ ] Module 1: Demo script for "what a well-configured OpenClaw can do"
- [ ] Module 3: Specific incident examples for security module (use real CVEs and ClawHavoc)
- [ ] Module 4: Pre-workshop setup guide for participants (Node install, API keys, messaging account)
- [ ] Module 6: Vetted skill recommendations for workshop exercises
- [ ] Module 8: 90-day plan template
- [ ] Participant handout / quick reference card
- [ ] Pricing and logistics
- [ ] All exercise briefs

## Questions for Justin

- [ ] Which messaging platforms are you currently using with OpenClaw?
- [ ] Which models are you running? What's your API spend?
- [ ] What agents do you currently have configured?
- [ ] What's the wonkiest part of your current setup?
- [ ] What do you want your OpenClaw to do that it doesn't currently?
- [ ] Who is the ideal workshop participant? Title, industry, technical comfort?
- [ ] Pricing range for the workshop?
- [ ] Do you have a venue/format preference?
- [ ] Timeline: when do you want to deliver the first workshop?
