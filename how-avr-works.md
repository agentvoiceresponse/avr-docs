---
title: How AVR Works
description: Understanding the core concepts and architecture of AVR
published: true
date: 2025-09-02T14:36:20.636Z
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

![architecture-avr.png](/images/architecture/architecture-avr.png)

### 1) Audio Ingestion (Asterisk → AVR Core)
1. The call hits your dialplan (e.g., `exten => 5001,...`), which:
   - **Answers** the call.
   - **Generates a UUID** (e.g., with `uuidgen`).
   - **Opens an AudioSocket** to AVR Core:  
     `AudioSocket(${UUID}, IP_AVR_CORE:PORT_AVR_CORE)`
2. Asterisk streams the caller’s **audio frames** to AVR Core in real time (typically narrowband telephony audio; exact codec depends on your PBX config).

**What AVR Core does:**
- Accepts the TCP AudioSocket connection.
- Normalizes audio if needed (codec/rate).
- **Segments audio into chunks** (streaming) and forwards them to the configured **ASR** over HTTP.

### 2) Transcription (AVR Core → ASR)
1. AVR Core **streams audio chunks** to the ASR at `ASR_URL`.
2. The ASR produces **partial results** (interim text) and **final results** (stabilized text).
3. AVR Core:
   - Buffers partials for responsiveness.
   - Emits a **final transcript segment** when the ASR marks it as finalized (e.g., end of user utterance).

> The **final transcript** is the trigger to move to the LLM step.

### 3) Reasoning/Response (AVR Core → LLM)
1. AVR Core forwards the **final transcript** (plus any conversation context) to the **LLM** at `LLM_URL`.  
   - Provider-agnostic (OpenAI, Anthropic, OpenRouter, etc.).  
   - **Example note**: “The response from **Anthropic** is then sent to a TTS engine…”
2. The LLM streams back the **assistant response** (text). AVR Core:
   - Streams partial tokens (if supported) or
   - Waits for the complete text, depending on the integration and configuration.

### 4) Voice Rendering (AVR Core → TTS)
1. AVR Core sends the LLM response text to **TTS** at `TTS_URL` (streaming).
2. TTS returns **audio frames** (the spoken reply).
3. AVR Core **pipes the audio back** over the existing AudioSocket to Asterisk, so the **caller hears the response** with minimal latency.

## Alternative Low-Latency Path (STS)

If you configure `STS_URL`, AVR Core will **bypass ASR/LLM/TTS** and use a **single Speech-to-Speech** service:

- **Speech In** → **STS** → **Speech Out**
- This can reduce latency and improve conversational “flow” by avoiding multiple hops.

![architecture-sts.png](/images/architecture/architecture-sts.png)

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


