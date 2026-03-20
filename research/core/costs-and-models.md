# OpenClaw Costs and Model Economics

> Last updated: 2026-03-20 | Sources: docs.openclaw.ai, GitHub issues, community reports, API pricing pages

## OpenClaw Is Free. The Models Are Not.

OpenClaw itself: $0 (MIT licence). All costs come from model API usage and infrastructure.

## Real-World Cost Ranges (AUD)

| Usage Level | Monthly (AUD) | Notes |
|---|---|---|
| Light personal (GPT-4.1-mini, optimised) | $19-24 | Daily briefings, occasional tasks |
| Light personal (Sonnet, optimised) | $56-71 | Heartbeat + moderate task load |
| Heavy personal (Opus) | $292-542 | Multiple agents, browser automation |
| Small team, 5 users (mixed models) | $330-750 | Shared gateway, mixed model routing |
| Uncontrolled power user | $850-5,100+ | Cautionary tales, not typical |

One tech blogger reported 1.8M tokens in a month with a $3,600 USD bill. Another hit $623 USD/month. These are what happens without cost controls.

## Heartbeat Cost (The Big Number)

Default heartbeat fires every 30 minutes = 48 calls/day.

**Without optimisation (default config):**

| Model | Per Day (AUD) | Per Month (AUD) |
|---|---|---|
| Claude Opus 4.6 | ~$4.05 | ~$122 |
| Claude Sonnet 4.6 | ~$2.43 | ~$72 |
| GPT-4.1 | ~$1.42 | ~$43 |
| GPT-4.1-mini | ~$0.28 | ~$8.50 |

**With isolatedSession + lightContext (2-5K tokens/run):**

| Model | Per Day (AUD) | Per Month (AUD) |
|---|---|---|
| Claude Opus 4.6 | ~$1.70 | ~$51 |
| Claude Sonnet 4.6 | ~$1.02 | ~$31 |
| GPT-4.1 | ~$0.57 | ~$17 |
| GPT-4.1-mini | ~$0.06 | ~$4.25 |

### CRITICAL BUG: lightContext May Be Broken

As of OpenClaw v2026.3.8, there is a [known bug](https://github.com/openclaw/openclaw/issues/43767) where `lightContext: true` is completely ignored and full agent context is loaded regardless. This means the cost optimisation may not actually work in the latest version. Verify on your version before relying on it.

## Model Pricing Reference (Per Million Tokens, USD)

| Model | Input | Output | Best For |
|---|---|---|---|
| Claude Opus 4.6 | $5.00 | $25.00 | Complex reasoning, highest quality |
| Claude Sonnet 4.6 | $3.00 | $15.00 | Balanced quality/cost |
| GPT-4.1 | $2.00 | $8.00 | Good reasoning, lower cost |
| GPT-4.1-mini | $0.40 | $1.60 | Budget tasks, routing, heartbeat |
| Local models (Ollama) | $0 | $0 | Privacy, zero API cost |

**Key optimisation:** Route tasks by model tier. Use mini for simple tasks (heartbeat, routing), flagship for complex reasoning. Can cut costs 60-80%.

## Hidden Costs

| Cost | Range |
|---|---|
| VPS/server (if not running locally) | $7-28/month |
| Docker hosting | $0-14/month |
| Tailscale (if using for remote access) | Free tier available |
| Multiple API provider accounts | Variable minimums |
| Time: initial setup | 4-8 hours (community estimate) |
| Time: ongoing maintenance | 1-2 hours/week |
| Time: security auditing and skill vetting | Ongoing |

## Cost Control Strategies

1. **Use cheaper models for heartbeat** — GPT-4.1-mini at $4.25/month vs Opus at $122/month
2. **isolatedSession + lightContext** — 20-50x reduction (when the bug is fixed)
3. **Active hours** — Don't run heartbeat while you sleep
4. **Longer intervals** — 60m = half the cost of 30m
5. **Empty HEARTBEAT.md** — Skips the run entirely (zero cost)
6. **Model fallbacks** — Expensive primary with cheap fallback
7. **Session maintenance** — Prune old sessions, set maxEntries
8. **Disable unused skills** — Each adds ~24 tokens to system prompt
9. **Spend alerts** — Monitor via `/usage` command

## Comparison to Alternatives

| Alternative | Monthly Cost | What You Get |
|---|---|---|
| Part-time VA (10 hrs/week) | $1,600+ | Human judgment, limited hours |
| Zapier Pro | $107 | 750 tasks/month, no autonomy |
| Microsoft Copilot | $43/user | Enterprise integration, limited agency |
| OpenClaw (optimised) | $19-71 | 24/7 autonomous agent |
