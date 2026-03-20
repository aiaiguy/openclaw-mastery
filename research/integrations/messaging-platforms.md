# OpenClaw Messaging Platform Integrations

> Last updated: 2026-03-20 | Source: docs.openclaw.ai

## Overview

OpenClaw is a self-hosted gateway (Node.js) connecting messaging platforms to AI agents. Supports **24+ channels** simultaneously. All channels share common access control: `dmPolicy` (pairing/allowlist/open/disabled) and `groupPolicy` (open/allowlist/disabled).

Config: `~/.openclaw/openclaw.json` (JSON5 format).

---

## Platform Comparison

| Feature | WhatsApp | Slack | Telegram | Discord | Signal | iMessage |
|---------|----------|-------|----------|---------|--------|----------|
| Setup Difficulty | Medium (QR) | Medium (app config) | Easy (BotFather) | Medium (dev portal) | Hard (signal-cli + Java) | Medium (Mac required) |
| Group Support | Yes | Yes | Yes (+ forums/topics) | Yes (+ forums/threads/voice) | Yes | Yes |
| Media Cap | 50MB | 20MB | 100MB | Varies | 8MB | 16MB |
| Streaming | No | Yes (native) | Yes (edit preview) | Yes (edit preview) | No | No |
| Interactive UI | Reactions only | Block Kit buttons/selects | Inline buttons | Components v2 (full) | Reactions | Basic |
| Voice | Voice notes | No | Voice notes | Voice channels + TTS | No | No |
| Multi-Account | Yes | Yes | Yes | Yes | Yes | Yes |
| Best For | Personal assistant, client comms | Team/workspace | Quick setup, solo use | Community/team, voice | Privacy-focused | Apple ecosystem |

**Also supported (not detailed here):** Google Chat, Microsoft Teams, IRC, Matrix, Feishu, LINE, Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon, Twitch, Zalo, WebChat.

---

## WhatsApp

**Technology:** Baileys library (WhatsApp Web protocol). Production-ready.

**Setup:**
1. Configure access policy in `openclaw.json`
2. Link via QR: `openclaw channels login --channel whatsapp`
3. Start gateway: `openclaw gateway`
4. Approve pairings if using pairing mode (codes expire after 1 hour, max 3 pending)

**Business vs Personal:**
- Dedicated number recommended for clearer routing
- Personal number supported with self-chat protections (skips read receipts for self-chat turns)
- Uses Baileys (Web protocol), not official WhatsApp Business API. No Twilio.

**Group Chat:** Mentions required by default. Up to 50 prior messages buffered as context. `/activation mention` or `/activation always` to toggle.

**Media:** Images, video, audio (PTT voice notes), documents. 50MB cap. Text chunked at 4,000 chars.

**Known Issues:**
- Bun runtime incompatible. Must use Node.js.
- Can experience disconnect loops. Fix: re-link via QR, run `openclaw doctor`
- Status/broadcast chats automatically ignored

---

## Slack

**Technology:** Bolt SDK. Two deployment modes.

**Socket Mode (default):**
1. Create Slack app with Socket Mode enabled
2. Generate App Token (`xapp-...`) with `connections:write` scope
3. Generate Bot Token (`xoxb-...`)
4. Subscribe to events: `app_mention`, `message.*`, `reaction_added/removed`
5. Configure tokens, launch gateway

**HTTP Events API:** Webhook mode for more complex deployments. Supports multi-account with distinct `webhookPath` values.

**Thread Handling:** Reply threading via `replyToMode` (off/first/all). Manual tags: `[[reply_to_current]]` and `[[reply_to:<id>]]`.

**Slash Commands:** Require `channels.slack.commands.native: true`. `/status` is reserved, use `/agentstatus`.

**Interactive UI:** Block Kit directives: `[[slack_buttons: Approve:approve, Reject:reject]]` and `[[slack_select: Choose a target | Option1:value1]]`. Slack-specific, won't work on other channels.

**Streaming:** Native Slack streaming via `chat.startStream/appendStream/stopStream`. Requires "Agents and AI Apps" enablement + `assistant:write` scope.

---

## Telegram

**Technology:** grammY (Bot API). Fastest setup of all channels.

**Setup:**
1. Chat with `@BotFather`, run `/newbot`
2. Save generated token
3. Configure via `channels.telegram.botToken` or `TELEGRAM_BOT_TOKEN` env var
4. Start gateway

**Forum/Topic Support:** Each topic can route to a different agent via `agentId`.

**Media:** Audio, video, stickers (static WEBP processed, animated TGS/WEBM skipped). 100MB cap. Sticker descriptions cached locally.

**Streaming:** `off`, `partial` (default, edits preview in-place), `block`, or `progress` modes.

**Execution Approvals:** Approval routing for tool execution. Approvers see buttons or `/approve` commands.

**Known Issues:**
- Disable Privacy Mode via BotFather if bot needs to see all group messages. After toggling, remove and re-add the bot.
- IPv6 resolution can cause failures. Force IPv4 with `autoSelectFamily: false`.

---

## Discord

**Technology:** discord.js with Gateway and Discord Bot API.

**Setup:**
1. Create application at Discord Developer Portal
2. Enable privileged intents: Message Content (mandatory), Server Members (recommended)
3. Generate invite URL with scopes `bot` + `applications.commands`
4. Set permissions: View Channels, Send Messages, Read History, Embed Links, Attach Files, Add Reactions
5. Configure bot token, start gateway

**Voice Channels:** Supported. Control via `/vc join|leave|status`. Auto-join and TTS configuration.

**Interactive UI:** Discord Components v2: buttons, select menus, modals (up to 5 fields). Reusable components. Permission gating via `allowedUsers`.

**Multi-Account:** Multiple Discord bots via `channels.discord.accounts.*`. Role-based routing.

**Known Issues:**
- Message Content Intent must be enabled in Developer Portal
- `allowBots=true` can cause bot loops. Prefer `allowBots="mentions"`

---

## Signal

**Technology:** signal-cli via HTTP JSON-RPC. External CLI integration.

**Setup options:**
- QR link to existing Signal account
- SMS registration with dedicated bot number

**Prerequisites:** Linux server (Ubuntu 24 tested), signal-cli, Java, phone number, browser for captcha.

**E2E Encryption:** Handled by signal-cli locally. Messages decrypted on your machine before reaching OpenClaw. No cloud intermediary.

**Critical Warning:** Registering a phone number with signal-cli **can de-authenticate your main Signal app** for that number. Use a dedicated bot number.

**Limitations:** No native mentions, no group read receipts, 8MB media cap, signal-cli requires frequent updates.

---

## Email

**No native email channel** as of March 2026.

Workarounds:
- **Gmail Pub/Sub** as webhook trigger for inbound
- **Skills** for email automation (Gmail tools via ClawHub)
- **Composio** for managed email API access

---

## Reliability Notes

- **Telegram** — Fastest setup, most reliable for solo use
- **WhatsApp** — Most popular but depends on Baileys (unofficial), can break with WhatsApp Web updates
- **Slack and Discord** — Best for team/workspace deployments
- **Signal** — Most maintenance (signal-cli updates)
- **iMessage** — Requires Mac running at all times
