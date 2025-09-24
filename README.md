# On-call Assistant Bot (n8n Template)

Turn noisy Slack alerts into an MCP-powered **On-call Assistant** that classifies alerts as **noise** or **signal**, posts rich context back to Slack, and can **auto-mute** noisy monitors in Observe.

---

## How it works

![On-call Assistant Bot Workflow](/images/n8n_workflow.png)

- **Slack Trigger** → listens to channel messages and `@mentions`  
- **Normalizer** → converts Slack (or webhook) payloads into a clean `alert` object  
- **AI Agent (Claude + Observe MCP)** → pulls logs/metrics/traces and builds an investigation worksheet URL  
- **Noise branch** → POSTs `monitor-mute-rules` to Observe to mute noisy monitors  
- **Threaded Slack replies** → clear verdict + links + next steps, or a mute confirmation

---

## Video: The Problem It Solves

No more 2AM fire drills for non-issues. This bot helps on-call engineers focus on **real** incidents.

▶️ **Watch on YouTube:** https://youtu.be/6g7A-aDb280

[![On-call Assistant Bot Video](https://img.youtube.com/vi/6g7A-aDb280/0.jpg)](https://youtu.be/6g7A-aDb280)

---

## Prerequisites

- **n8n Cloud** (built and tested on n8n Cloud)  
- A **Slack App (Bot)** in your workspace  
- An **Observe** tenant + API token (for MCP + mute API)

> 🛡️ This repo and the provided template **do not** contain secrets. You’ll attach credentials inside n8n.

---

## Quick Start

### 1) Create your Slack App & Bot

1. Go to **[api.slack.com/apps](https://api.slack.com/apps)** → **Create New App** → *From scratch*  
2. **App name:** `On-call Assistant Bot` ; **Default username:** `on_call_assistant_bot`  
3. **OAuth Scopes (Bot Token):**
   - Required: `chat:write`, `channels:history`, `channels:read`, `app_mentions:read`
   - Optional (if needed): `chat:write.public`, `users:read`, `groups:history`, `im:history`, `mpim:history`
4. **Install to Workspace** → copy the **Bot User OAuth Token** (`xoxb-…`) and **Signing Secret**

### 2) Configure Event Subscriptions

1. Enable **Event Subscriptions**  
2. **Request URL** = the Slack Trigger node’s **Production URL** (shown in n8n after import)  
   - Example: `https://{{N8N_HOST}}/webhook/<workflow-id>`  
3. **Subscribe to bot events:**  
   - `message.channels`  
   - `app_mention`  
4. Save and **Reinstall** if prompted  
5. `/invite @on_call_assistant_bot` into your target channel

---

## Slack Bot Configuration (Reference)

When configuring your Slack App, ensure:

### Event Subscriptions

Enable **Event Subscriptions** and set the **Request URL** to your n8n Slack Trigger node’s Production URL: https://{{N8N_HOST}}/webhook/<workflow-id>


Subscribed bot events:
- `message.channels`
- `app_mention`

![Slack Event Subscriptions Example](/images/slack-bot-event-subscriptions.png)

---

### OAuth Scopes

Required Bot Token Scopes:
- `app_mentions:read`
- `channels:history`
- `channels:read`
- `chat:write`

Optional (depending on your use case):
- `chat:write.public`
- `users:read`
- `groups:history`
- `im:history`
- `mpim:history`

![Slack Bot Scopes Example](/images/slack-bot-scopes.png)

---

> 💡 Tip: The screenshots above show a working configuration. Adjust scopes or events depending on whether you want the bot in private channels, group DMs, or just public channels.

---

## Import the n8n Template

- In n8n Cloud, **Import** `On-call-Assistant-Bot-Flow.template.json`  
- You should see something like:

![On-call Assistant Bot Flow in n8n Cloud](/images/n8n_cloud_screenshot.png)

### Set Placeholders & Credentials in n8n

Replace the following placeholders in node parameters (don’t hardcode secrets):

| Placeholder | What it is | Example |
|---|---|---|
| `{{SLACK_CHANNEL_ID}}` | The channel ID where the bot listens/replies | `C0123ABCDEF` |
| `{{SLACK_BOT_USER_ID}}` | Your bot user’s ID | `U0456GHIJKL` |
| `{{OBSERVE_HOST}}` | Your Observe host | `https://123456789012.observeinc.com` |
| `{{ANTHROPIC_MODEL}}` | Anthropic model alias/name used by your n8n cred | `claude-3-5-sonnet` |

Attach credentials in **n8n → Credentials** and reference them from nodes:

- **Slack**: Bot Token (`xoxb-…`) + **Signing Secret**  
- **HTTP / MCP to Observe**: Header Auth (e.g., `Authorization: Bearer <TOKEN>`) and base URL `{{OBSERVE_HOST}}`  
- **HTTP / Mute API**: Bearer token for `monitor-mute-rules` endpoint

> Tip: Use n8n **Environment Variables** or **Credentials**—avoid literals inside the workflow.

---

## Activate & Test

- **Activate** the workflow  
- Mention the bot in your channel or fire a test alert payload  
- Expect a threaded reply with verdict + links (and a :mute: confirmation when classified as **noise**)

---

## Examples

### Example 1 — **Signal Alert**

Payment Service Error Rate Alert triggered.  
The bot classifies it as **SIGNAL** with severity **ERROR**, and posts context + suggested next steps.

![Signal Example](/images/signal.png)

### Example 2 — **Noise Alert**

Frontend High Traffic Alert triggered.  
The bot classifies it as **NOISE**, explains why, and **auto-mutes** the related monitor.

![Noise Example](/images/noise.png)

---

## Customize

- **Mute duration** – change `durSec` in *Build Mute Body* (default: `3600`)  
- **Slack message format** – tweak message templates / expressions  
- **Evidence gate** – require specific `(mcp:*)` signals to vote **signal**  
- **Dedup mutes** – add an **n8n Data Store** keyed by `mute:<monitorId>` to avoid repeat mutes

---

## Security Notes

- This template ships **without** secrets. Add your own credentials in n8n  
- Keep tenant IDs, channel IDs, and user IDs out of version control—use placeholders  
- Review Slack scopes and HTTP node permissions for least privilege  
- Consider rotating API tokens and setting short mute durations by default

---

## Files in this repo

- `On-call-Assistant-Bot-Flow.template.json` – sanitized n8n template to import  
- `/images/*` – screenshots used in this README

---

## Troubleshooting

- **Slack “Request URL failed validation”** → Ensure the Slack Trigger node is **Production URL** and publicly reachable  
- **Bot doesn’t reply** → Check scopes, channel membership (`/invite`), and that the workflow is **Active**  
- **Observe calls fail** → Verify `{{OBSERVE_HOST}}`, token, and required headers; test the HTTP nodes directly in n8n  
- **Mutes not applied** → Confirm monitor IDs and that your token has permission for `monitor-mute-rules`

---

Happy on-call! :pager: :robot:
