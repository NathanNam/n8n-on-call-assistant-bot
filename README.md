# On-call Assistant Bot (n8n Template)

This workflow turns Slack alerts into a triaged, MCP-powered **On-call Assistant** that can classify alerts as **noise** or **signal**, post rich context back to Slack, and **auto-mute** noisy monitors in Observe.

---

## Workflow Overview

Here’s how the workflow looks in n8n:

![On-call Assistant Bot Workflow](/images/n8n_workflow.png)

- **Slack Trigger** → listens to channel messages + @mentions  
- **Normalizer** → converts Slack webhook payloads into a clean `alert` object  
- **AI Agent (Claude + Observe MCP)** → fetches logs/metrics/traces & builds an investigation worksheet URL  
- **Noise branch** → POSTs `monitor-mute-rules` to Observe to mute the monitor  
- **Threaded Slack replies** → clear verdict + links + next steps, or a mute confirmation  

---

## Short Video: The Problem It Solves

We created a short video that captures the problem the **On-call Assistant Bot** solves:  
helping engineers avoid noisy 2AM alerts and focus on real incidents.

▶️ [Watch on YouTube](https://youtu.be/6g7A-aDb280)

[![On-call Assistant Bot Video](https://img.youtube.com/vi/6g7A-aDb280/0.jpg)](https://youtu.be/6g7A-aDb280)

---

## Prereqs
- **n8n** (Cloud or self-hosted)  
- A **Slack App** (Bot) for your workspace  
- **Observe** tenant + API token  

---

## Setup

### 1) Create Slack App & Bot
1. Go to [Slack API Apps](https://api.slack.com/apps) → **Create New App** → From scratch.  
2. **App name:** `On-call Assistant Bot` ; **Default username:** `on_call_assistant_bot`.  
3. **Bot Scopes**:  
   - `chat:write`, `channels:history`, `channels:read`, `app_mentions:read`  
   - Optional: `users:read`; private chats: `groups:history`, `im:history`, `mpim:history`  
4. **Install to Workspace** → copy Bot User OAuth Token (xoxb-…).  

### 2) Event Subscriptions
1. Enable **Event Subscriptions**.  
2. **Request URL** = Slack Trigger node’s **Production URL**.  
3. **Subscribe to bot events**: `message.channels`, `app_mention`.  
4. **Save** → Reinstall if prompted.  
5. `/invite @on_call_assistant_bot` to your channel.  

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

## Examples

### Example 1 — **Signal Alert**

Payment Service Error Rate Alert triggered.  
The bot triages it as **SIGNAL**, with severity **ERROR**, and provides context + next steps.

![Signal Example](/images/signal.png)

---

### Example 2 — **Noise Alert**

Frontend High Traffic Alert triggered.  
The bot triages it as **NOISE**, explains why, and automatically mutes the monitor.

![Noise Example](/images/noise.png)

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

---

Happy on-call! :pager: :robot:
