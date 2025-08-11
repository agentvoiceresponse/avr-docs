---
title: Deepgram
description: Deepgram is an AI-powered speech platform providing both Automatic Speech Recognition (ASR) and Text-to-Speech (TTS) capabilities.
published: true
date: 2025-08-11T10:20:30.335Z
tags: asr, tts
editor: markdown
dateCreated: 2025-08-11T10:20:30.335Z
---

# Configuring Deepgram in AVR

## Overview
[Deepgram](https://deepgram.com/) is an AI-powered speech platform providing both **Automatic Speech Recognition (ASR)** and **Text-to-Speech (TTS)** capabilities.

**ASR Features:**
- Real-time and batch transcription.
- High accuracy with advanced models like *Nova-2*.
- Support for over 36 languages and dialects.
- Low latency (under 300 ms for streaming).
- Options for diarization, keyword boosting, and filler word removal.

**TTS Features:**
- Natural-sounding, low-latency voice synthesis.
- Multiple voices and styles across supported languages.
- Configurable models, voices, and versions.

---

## 1. Deepgram ASR Setup

### Repository
- GitHub: [avr-asr-deepgram](https://github.com/agentvoiceresponse/avr-asr-deepgram)

### Environment Variables
| Variable                       | Description                                               | Example Value            |
|--------------------------------|-----------------------------------------------------------|--------------------------|
| `PORT`                         | Port where the ASR module listens                         | `6010`                   |
| `DEEPGRAM_API_KEY`             | Your Deepgram API key                                     | `abc123xyz...`           |
| `SPEECH_RECOGNITION_LANGUAGE`  | Target language code for transcription                    | `en-US`                  |
| `SPEECH_RECOGNITION_MODEL`     | Deepgram model name (including version, if applicable)    | `nova-2-phonecall`       |

**Example `.env` section:**
```
DEEPGRAM_API_KEY=your_deepgram_api_key_here
SPEECH_RECOGNITION_LANGUAGE=en-US
SPEECH_RECOGNITION_MODEL=nova-2-phonecall
PORT=6010
```
Docker Compose Configuration:

```
avr-asr-deepgram:
  image: agentvoiceresponse/avr-asr-deepgram
  platform: linux/x86_64
  container_name: avr-asr-deepgram
  restart: always
  environment:
    - PORT=6010
    - DEEPGRAM_API_KEY=$DEEPGRAM_API_KEY
    - SPEECH_RECOGNITION_LANGUAGE=en-US
    - SPEECH_RECOGNITION_MODEL=nova-2-phonecall
  networks:
    - avr
```

⸻

2. Deepgram TTS Setup

Repository
	•	GitHub: avr-tts-deepgram

Environment Variables

Variable	Description	Example Value
PORT	Port where the TTS module listens	6011
DEEPGRAM_API_KEY	Your Deepgram API key	abc123xyz...
DEEPGRAM_TTS_MODEL	Deepgram TTS model in format [model]-[voice]-[language]-[version]	aura-arabella-en-xxhifi

Example .env section:

DEEPGRAM_API_KEY=your_deepgram_api_key_here
DEEPGRAM_TTS_MODEL=aura-arabella-en-xxhifi
PORT=6011

Docker Compose Configuration:

avr-tts-deepgram:
  image: agentvoiceresponse/avr-tts-deepgram
  platform: linux/x86_64
  container_name: avr-tts-deepgram
  restart: always
  environment:
    - PORT=6011
    - DEEPGRAM_API_KEY=$DEEPGRAM_API_KEY
    - DEEPGRAM_TTS_MODEL=aura-arabella-en-xxhifi
  networks:
    - avr


⸻

How AVR Uses Deepgram
	1.	ASR Module: AVR Core streams audio to Deepgram ASR, receives transcribed text, and passes it to the LLM.
	2.	TTS Module: AVR Core sends the LLM’s text response to Deepgram TTS, which returns audio to Asterisk or the calling application.

Do you want me to **append a list of available Deepgram ASR and TTS models** directly into this Markdown so users can copy-paste correct values? That would make the config section much more complete.