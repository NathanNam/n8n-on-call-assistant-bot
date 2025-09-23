
# On-call Assistant Bot (n8n Template)

This workflow turns Slack alerts into a triaged, MCP-powered **On-call Assistant** that can classify alerts as **noise** or **signal**, post rich context back to Slack, and **auto-mute** noisy monitors in Observe.

## What you get
- **Slack Trigger** → listens to channel messages + @mentions
- **Normalizer** → converts Slack webhook payloads into a clean `alert` object
- **AI Agent (Claude + Observe MCP)** → fetches logs/metrics/traces & builds an investigation worksheet URL
- **Noise branch** → POSTs `monitor-mute-rules` to Observe to mute the monitor
- **Threaded Slack replies** → clear verdict + links + next steps, or a mute confirmation

---

## Prereqs
- **n8n** (Cloud or self-hosted)
- A **Slack App** (Bot) for your workspace
- **Observe** tenant + API token

---

## Setup

### 1) Create Slack App & Bot
1. https://api.slack.com/apps → **Create New App** → From scratch.
2. **App name:** `On-call Assistant Bot` ; **Default username:** `on_call_assistant_bot`.
3. **Bot Scopes**: `chat:write`, `channels:history`, `channels:read`, `app_mentions:read`.
   - Optional: `users:read`; private chats: `groups:history`, `im:history`, `mpim:history`.
4. **Install to Workspace** → copy Bot User OAuth Token (xoxb-…).

### 2) Event Subscriptions
1. Enable **Event Subscriptions**.
2. **Request URL** = Slack Trigger node’s **Production URL**.
3. **Subscribe to bot events**: `message.channels`, `app_mention`.
4. **Save** → Reinstall if prompted; `/invite @on_call_assistant_bot` to your channel.

### 3) Import & Configure
- Import `On-call-Assistant-Bot-Flow.template.json` into n8n.
- Replace placeholders:
  - `<SLACK_CHANNEL_ID>` (channel detail pane)
  - `<SLACK_BOT_USER_ID>` (from bot profile)
  - `<OBSERVE_CUSTOMERID>` (numeric subdomain)
- Attach credentials (do **not** hardcode tokens in the workflow):
  - Slack OAuth (xoxb + Signing Secret)
  - HTTP Header Auth for MCP (`Authorization: Bearer <OBSERVE_CUSTOMERID> <TOKEN>`)
  - HTTP Bearer for mute POST

### 4) Activate & Test
- **Activate** the workflow.
- Mention the bot in your channel or fire a test alert.
- You should see a threaded reply with verdict + links (and a :mute: confirmation when `noise`).

---

## Customize
- **Mute duration**: change `durSec` in *Build Mute Body* (default 3600s).
- **Links shown**: tweak the Slack message expressions.
- **Evidence gate**: require `(mcp:*)` signals for `signal` in the AI normalizer.
- **Dedup mutes**: add an n8n Data Store step keyed by `mute:<monitorID>`.

---

## Security Notes
- This template contains **no secrets**. Attach your own credentials in n8n.
- Keep your tenant/channel/user IDs out of version control; use placeholders.

Happy on-call! :pager: :robot:
