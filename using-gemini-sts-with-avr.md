---
title: Using Gemini STS with AVR
description: 
published: true
date: 2026-02-08T12:29:28.028Z
tags: 
editor: markdown
dateCreated: 2025-09-02T13:38:04.526Z
---

# Using Gemini STS with AVR

<div align="center">
  <img src="/images/gemini/gemini-logo.png" alt="Gemini Logo" width="300"/>
</div>

**Gemini** is Google’s family of multimodal AI models. With the **Gemini Speech-to-Speech (STS)** integration, Agent Voice Response (AVR) can handle conversations where **speech input is directly transformed into speech output**, without requiring separate ASR (speech-to-text) and TTS (text-to-speech) components.

This approach significantly **reduces latency** and enables more **natural, human-like voice interactions**.



## Why Use Gemini STS?

- **End-to-End Speech Conversations** — Direct speech-to-speech transformation.
- **Low Latency** — Faster than traditional ASR + LLM + TTS pipelines.
- **Multilingual Support** — Supports multiple languages out of the box.
- **Instruction-Tuned** — Fully customizable system prompts.
- **Cloud-Native** — Runs on Google’s scalable and reliable infrastructure.



## Generate API Credentials

To connect AVR with Gemini, you need a Gemini API key:

1. Open **Google AI Studio**
2. Sign in with your Google account
3. Navigate to **API Keys**
4. Create a new API key
5. Copy and store it securely

You will use this key as `GEMINI_API_KEY` in your Docker environment.



## Repository

- GitHub: https://github.com/agentvoiceresponse/avr-sts-gemini



## Environment Variables

| Variable | Description | Example Value |
|--------|-------------|---------------|
| `PORT` | Port on which the Gemini STS service runs | `6037` |
| `GEMINI_API_KEY` | API Key from Google AI Studio | `AIza...` |
| `GEMINI_MODEL` | Gemini model ID to use | `gemini-2.5-flash-preview-native-audio-dialog` |
| `GEMINI_INSTRUCTIONS` | System prompt for the voice assistant | `"You are a helpful assistant."` |
| `GEMINI_URL_INSTRUCTIONS` | URL to fetch dynamic instructions | `https://your-api.com/instructions` |
| `GEMINI_FILE_INSTRUCTIONS` | Path to local instruction file | `./instructions.txt` |

### Gemini thinking settings

We’ve added support for the following Gemini settings:

- `GEMINI_THINKING_LEVEL=MINIMAL`
- `GEMINI_THINKING_BUDGET=0`

More details here 👉 https://ai.google.dev/gemini-api/docs/thinking?hl=en

Supported values for `GEMINI_THINKING_LEVEL`:

- `THINKING_LEVEL_UNSPECIFIED`
- `LOW`
- `MEDIUM`
- `HIGH`
- `MINIMAL`

`GEMINI_THINKING_BUDGET`:

- `0` → turn off thinking
- `-1` → enable dynamic thinking


## Docker Setup

Add the following service to your `docker-compose.yml`:

```yaml
avr-sts-gemini:
  image: agentvoiceresponse/avr-sts-gemini
  platform: linux/x86_64
  container_name: avr-sts-gemini
  restart: always
  environment:
    - PORT=6037
    - GEMINI_API_KEY=$GEMINI_API_KEY
    - GEMINI_MODEL=$GEMINI_MODEL
    - GEMINI_INSTRUCTIONS=$GEMINI_INSTRUCTIONS
  networks:
    - avr
```



## Integration with AVR Core

Configure **avr-core** to use Gemini STS by setting `STS_URL`:

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    - STS_URL=ws://avr-sts-gemini:6037
  ports:
    - 5001:5001
  networks:
    - avr
```

> ⚠️ When `STS_URL` is configured, `ASR_URL`, `LLM_URL`, and `TTS_URL` must be commented out.



## Instruction Loading Methods

The Gemini STS integration supports **multiple instruction sources** with a clear priority order.

### 1. Environment Variable (Highest Priority)

```env
GEMINI_INSTRUCTIONS="You are a specialized customer service agent for a tech company. Always be polite and helpful."
```

If set, this overrides all other instruction sources.



### 2. Web Service (Medium Priority)

```env
GEMINI_URL_INSTRUCTIONS="https://your-api.com/instructions"
```

Expected response format:

```json
{
  "system": "You are a helpful assistant that provides technical support."
}
```

The request will include the call UUID as an HTTP header:

```
X-AVR-UUID: <call-uuid>
```

This enables **dynamic, per-call instruction generation**.



### 3. File-Based Instructions (Lowest Priority)

```env
GEMINI_FILE_INSTRUCTIONS="./instructions.txt"
```

The file should contain plain text instructions.



### Instruction Priority Order

1. `GEMINI_INSTRUCTIONS`
2. `GEMINI_URL_INSTRUCTIONS`
3. `GEMINI_FILE_INSTRUCTIONS`
4. Default instructions (fallback)



## Example in AVR Infra

A ready-to-use example is available in the **avr-infra** repository:

- `docker-compose-gemini.yml` — Example #10

https://github.com/agentvoiceresponse/avr-infra



## Performance Notes

- Optimized for real-time, low-latency conversations
- Requires a stable internet connection
- VAD is handled natively by Gemini
- AVR Core VAD and `INTERRUPT_LISTENING` are ignored in STS mode