---
title: How AVR Works
description: Understanding the core concepts and architecture of AVR
published: true
date: 2025-08-31T19:54:58.178Z
tags: 
editor: markdown
dateCreated: 2025-08-06T17:06:33.271Z
---

# How AVR Works

The AVR Infrastructure project provides a complete, modular deployment environment for the Agent Voice Response system. It allows you to launch the AVR Core, ASR, LLM, and TTS services, all integrated with an Asterisk PBX using the AudioSocket protocol.

This setup supports a wide range of providers—including cloud services like OpenAI, Deepgram, Google, ElevenLabs, Anthropic **and local/open-source providers like Vosk, Kokoro, CoquiTTS, Ollama**—and can be customized with Docker Compose.

## Prerequisites

Before starting, ensure the following tools and credentials are ready:

- Docker and Docker Compose installed  
- Valid API keys for services you plan to use (e.g., OpenAI, Google, Deepgram)  
- (Optional) SIP client installed (e.g., Telphone, MicroSIP, GNOME Calls) for voice testing  
- (Optional) Local ASR/TTS/LLM services configured if using Vosk, Kokoro, CoquiTTS, or Ollama  

## Architecture Summary

AVR follows a modular design:

![asr-llm-tts.png](/asr-llm-tts.png)

Optionally, you can use a Speech-to-Speech (STS) provider like OpenAI Realtime, Ultravox, or other compatible engines to replace the ASR–LLM–TTS chain:

![sts.png](/sts.png)

## Your First Agent in Under 5 Minutes

Use one of the preconfigured `docker-compose-*.yml` files to deploy AVR with your preferred providers.

### Step-by-step:

1. Clone the repository:

```console
git clone https://github.com/agentvoiceresponse/avr-infra
cd avr-infra
```


2. Copy .env.example to .env:

```console
cp .env.example .env
```

3. Select a compose file:

For example, to run Deepgram + OpenAI:

```console
docker-compose -f docker-compose-openai.yml up -d
```

Or, for local/open-source providers like Vosk:

```console
docker-compose -f docker-compose-vosk.yml up -d
```

4. Edit .env with your API keys or local service URLs. Example:

```env
# Cloud providers
DEEPGRAM_API_KEY=your_key
OPENAI_API_KEY=your_key
OPENAI_MODEL=gpt-3.5-turbo

# Local providers

```

#### Advanced: STS-Only Providers

Some providers (like OpenAI Realtime or Ultravox) offer Speech-to-Speech (STS) processing that bundles ASR, LLM, and TTS.

In this case, only set:

```env
STS_URL=http://avr-sts-provider:port
```

See `docker-compose-deepgram.yml` and `docker-compose-ultravox.yml` for examples.


