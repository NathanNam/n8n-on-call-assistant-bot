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

## Prereqs
- **n8n** (Cloud or self-hosted)  
- A **Slack App** (Bot) for your workspace  
- **Observe** tenant + API token  

---

## Setup

### 1) Create Slack App & Bot
1. Go to [Slack API Apps](https://api.slack.com/apps) → **Create New App** → From scratch.  
2. **App name:** `On-call Assistant Bot` ; **Default username:** `on_call_assist