---
title: Using AVR with N8N
description: 
published: true
date: 2026-04-24T07:26:08.435Z
tags: 
editor: markdown
dateCreated: 2025-08-26T11:02:33.190Z
---

# Using AVR with n8n (AI Workflow Integration)

<div align="center">
  <img src="/images/n8n/n8n.png" alt="FreePBX" width="300"/>
</div>

Integrating AVR with [n8n](https://n8n.io) allows you to build **AI-powered voicebots** with visual workflows and direct integration with AVR.  
This integration is powered by the [avr-llm-n8n](https://github.com/agentvoiceresponse/avr-llm-n8n) connector.

## 🚀 What's New (v1.1.0)

Starting from version **1.1.0**, the integration has been updated to support **streaming mode** using the n8n AI Agent capabilities.

- Native streaming responses (low latency, real-time)
- Support for both Chat Trigger and Webhook nodes
- Improved conversational experience

Docs: https://docs.n8n.io/workflows/streaming/

## Environment Variables

| Variable         | Description                                           | Example Value                                     |
|------------------|-------------------------------------------------------|---------------------------------------------------|
| `PUBLIC_CHAT_URL`| Your n8n public chat workflow endpoint                | `https://your-n8n-instance.com/webhook/chat`      |
| `PORT`           | Port where the AVR n8n connector will listen          | `6016`                                            |

Replace `your_n8n_public_chat_endpoint` with your actual n8n public chat workflow URL.

---

## Example docker-compose (AVR n8n connector)

```yaml
avr-llm-n8n:
  image: agentvoiceresponse/avr-llm-n8n
  platform: linux/x86_64
  container_name: avr-llm-n8n
  restart: always
  environment:
    - PORT=6016
    - PUBLIC_CHAT_URL=$PUBLIC_CHAT_URL
  networks:
    - avr
```

## Example with local n8n

In the avr-infra project, you can find a complete example of how to integrate n8n with AVR.
The compose file includes both the AVR n8n connector and a local instance of n8n:
```yaml
avr-n8n:
  image: n8nio/n8n:latest
  container_name: avr-n8n
  environment:
    - GENERIC_TIMEZONE=Europe/Amsterdam
    - NODE_ENV=production
    - N8N_SECURE_COOKIE=false
  ports:
    - 5678:5678
  volumes:
    - ./n8n:/home/node/.n8n
  networks:
    - avr
```

> If you already have an n8n installation (either in the cloud or on another server), you can comment out the avr-n8n section.
> {.is-info}
{.is-warning}


> If you use the local installation, first create an account by providing email, first name, last name, and password, then continue with the setup below.
![user.png](/images/n8n/user.png)
{.is-info}

## Step-by-step Setup AI Voicebot (Trigger: "When chat message received")

![workflow.png](/images/n8n/workflow.png)

### 1. Start with a Chat Trigger
In n8n, create a new workflow with a Chat Trigger node.
Enable “Make Chat Publicly Available” and copy the **Chat URL** to use in **PUBLIC_CHAT_URL**.
Add option "Set Response Mode" and select "Streaming".

![trigger.png](/images/n8n/trigger.png)

### 2.	Connect the Chat Trigger to an AI Agent node
Choose whether to use a Conversational Agent or a Tools Agent depending on your workflow needs.
Add option "Enable Streaming" and enable it.

![ai-agent.png](/images/n8n/ai-agent.png)

### 3.	Add your Chat Model
Insert an AI Chat Model node (e.g., OpenAI, Anthropic, etc.).
Configure temperature, max tokens, and model according to your application.

![openai-model.png](/images/n8n/openai-model.png)

### 4.	Enable Memory
Add a memory node (e.g., buffer memory) for context-aware conversations.
Use the chat session ID from the Chat Trigger as the session key.

![simple-memory.png](/images/n8n/simple-memory.png)

### 5.	Add Tools like SerpAPI

Extend your workflow with additional nodes like SerpAPI, databases, or custom APIs.

![serpapi-apikey.png](/images/n8n/serpapi-apikey.png)

![serpapi.png](/images/n8n/serpapi.png)


## Step-by-step Setup AI Voicebot (Trigger: "Webhook")
![webhook-workflow.png](/images/n8n/webhook-workflow.png)

### 1. Start with a Webhook trigger
In n8n, create a new workflow with a Webhook Trigger node.
Copy the **Production URL** to use in **PUBLIC_CHAT_URL**.
Set "Respond" as "Streaming".

![webhook-trigger.png](/images/n8n/webhook-trigger.png)

### 2.	Set params for AI Agent node
Insert an Set node and configure:
- sessionId = {{ $json.body.sessionId }}
- chatInput = {{ $json.body.chatInput }}

![webhook-params.png](/images/n8n/webhook-params.png)


### 2.	Connect the Chat Trigger to an AI Agent node
Choose whether to use a Conversational Agent or a Tools Agent depending on your workflow needs.
Add option "Enable Streaming" and enable it.

![ai-agent.png](/images/n8n/ai-agent.png)

### 3.	Add your Chat Model
Insert an AI Chat Model node (e.g., OpenAI, Anthropic, etc.).
Configure temperature, max tokens, and model according to your application.

![openai-model.png](/images/n8n/openai-model.png)

### 4.	Enable Memory
Add a memory node (e.g., buffer memory) for context-aware conversations.
Use the chat session ID from the Chat Trigger as the session key.

![simple-memory.png](/images/n8n/simple-memory.png)

### 5.	Add Tools like SerpAPI

Extend your workflow with additional nodes like SerpAPI, databases, or custom APIs.

![serpapi-apikey.png](/images/n8n/serpapi-apikey.png)

![serpapi.png](/images/n8n/serpapi.png)

