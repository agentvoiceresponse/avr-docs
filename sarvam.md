---
title: Sarvam ASR, LLM & TTS
description: 
published: true
date: 2026-02-06T23:21:06.383Z
tags: 
editor: markdown
dateCreated: 2026-02-06T23:20:16.091Z
---

# Sarvam

Sarvam is a high-performance AI provider optimized for **Indic (Indian) languages** and real-time voice applications.
Agent Voice Response (AVR) integrates Sarvam as a **full conversational stack**, covering:

* **ASR** – Real-time Speech-to-Text
* **LLM** – Chat / reasoning
* **TTS** – Low-latency Text-to-Speech

This makes Sarvam a strong choice for **Indian telephony, IVR replacement, and voicebot deployments**.

---

## Supported Capabilities

| Capability                  | Status |
| --------------------------- | ------ |
| Speech-to-Text (ASR)        | ✅      |
| Large Language Models (LLM) | ✅      |
| Text-to-Speech (TTS)        | ✅      |
| Streaming / Real-time       | ✅      |
| Telephony (8kHz PCM)        | ✅      |
| Indic Languages             | ✅      |

---

## Speech-to-Text (ASR)

The Sarvam ASR integration uses **Sarvam WebSocket APIs** to provide real-time transcription, optimized for **telephony audio**.

### Key Features

* Real-time streaming transcription
* WebSocket → SSE bridge
* Telephony-ready audio format
* Language hints support (Indic languages)

### AVR Endpoint

```
POST /speech-to-text-stream
```

* Input: raw audio stream
* Output: incremental transcription via **Server-Sent Events (SSE)**

### Audio Requirements

| Parameter   | Value                 |
| ----------- | --------------------- |
| Encoding    | s16le (PCM 16-bit LE) |
| Sample Rate | 8000 Hz               |
| Channels    | Mono                  |

### Environment Variables

```env
SARVAM_API_KEY=sk_...
SARVAM_WEBSOCKET_URL=wss://api.sarvam.ai/speech-to-text/ws
SARVAM_SPEECH_RECOGNITION_MODEL=saarika:v2.5
SARVAM_SPEECH_RECOGNITION_LANGUAGE=en-IN
PORT=6050
```

## Large Language Model (LLM)

The Sarvam LLM integration connects AVR to **Sarvam Chat Completions**, providing conversational reasoning optimized for Indic languages.

### Key Features

* Lightweight HTTP proxy
* Compatible with AVR dialogue pipeline
* Configurable system prompt
* Strong Hindi / Indic language support

### AVR Endpoint

```
POST /prompt-stream
```

### Request Example

```json
{
  "messages": [
    { "role": "user", "content": "Hello, how can you help me?" }
  ]
}
```

### Response Format

```json
{
  "type": "text",
  "content": "..."
}
```

### Environment Variables

```env
SARVAM_API_KEY=sk_...
SARVAM_MODEL=sarvam-m
SYSTEM_PROMPT="You are a helpful assistant."
PORT=6051
```

---

## Text-to-Speech (TTS)

The Sarvam TTS integration generates **real-time streamed audio**, suitable for **telephony and IVR systems**.

### Key Features

* Low-latency audio streaming
* Natural Indic voices
* Telephony-compatible WAV output

### AVR Endpoint

```
POST /text-to-speech-stream
```

### Request Example

```json
{
  "text": "Hello, how can I assist you today?"
}
```

### Audio Output Format

| Parameter   | Value      |
| ----------- | ---------- |
| Container   | WAV        |
| Sample Rate | 8000 Hz    |
| Encoding    | PCM 16-bit |
| Channels    | Mono       |

### Environment Variables

```env
SARVAM_API_KEY=sk_...
SARVAM_TTS_LANGUAGE=en-IN
SARVAM_TTS_SPEAKER=aditya
SARVAM_TTS_MODEL=bulbul:v3
SARVAM_TTS_TEMPERATURE=0.6
PORT=6052
```

---

## Full Docker Compose (ASR + LLM + TTS)

A complete Sarvam stack for Agent Voice Response:

```yaml
version: "3.9"

services:
  avr-asr-sarvam:
    image: agentvoiceresponse/avr-asr-sarvam
    container_name: avr-asr-sarvam
    env_file: .env
    ports:
      - "6050:6050"
    restart: unless-stopped

  avr-llm-sarvam:
    image: agentvoiceresponse/avr-llm-sarvam
    container_name: avr-llm-sarvam
    env_file: .env
    ports:
      - "6051:6051"
    restart: unless-stopped

  avr-tts-sarvam:
    image: agentvoiceresponse/avr-tts-sarvam
    container_name: avr-tts-sarvam
    env_file: .env
    ports:
      - "6052:6052"
    restart: unless-stopped
```

Start the stack:

```bash
docker compose up -d
```

---

## When to Use Sarvam

Sarvam is recommended when:

* Your voicebot operates in **India**
* **Indic languages** are required
* **Low-latency telephony audio** is critical
* You want a **single provider for ASR, LLM, and TTS**