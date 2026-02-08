---
title: Speechmatics STS
description: 
published: true
date: 2026-02-08T18:35:07.942Z
tags: 
editor: markdown
dateCreated: 2026-02-08T18:34:28.507Z
---

# Using Speechmatics STS with AVR

Speechmatics provides enterprise-grade speech recognition and synthesis.  
With the **Speech-to-Speech (STS)** integration, **Agent Voice Response (AVR)** can handle real-time conversations where speech is directly transformed into speech—without requiring separate ASR and TTS services.

This approach reduces latency and simplifies the overall architecture.

## Why use Speechmatics STS?

- **End-to-End Speech Conversations**  
  Direct speech-in → speech-out processing with no intermediate text pipeline.

- **Low Latency Streaming**  
  Optimized real-time audio streaming via WebSocket.

- **Production-Grade Quality**  
  High-accuracy ASR combined with natural-sounding TTS.

- **Flexible AI Control**  
  Uses OpenAI models for response generation while Speechmatics handles speech processing.

- **Native AVR Integration**  
  Fully compatible with AVR Core via the standard STS interface.

## Repository

- **GitHub**: https://github.com/agentvoiceresponse/avr-sts-speechmatics

## Environment Variables

Configure the Speechmatics STS container using the following environment variables:

```env
# Required
SPEECHMATICS_API_KEY=your_speechmatics_api_key
```

## Docker Setup

Add the Speechmatics STS service to your docker-compose.yml:

```yaml
avr-sts-speechmatics:
  image: agentvoiceresponse/avr-sts-speechmatics
  platform: linux/x86_64
  container_name: avr-sts-speechmatics
  restart: always
  environment:
    - SPEECHMATICS_API_KEY=${SPEECHMATICS_API_KEY}
    - PORT=6040
  networks:
    - avr
```

Integration with AVR Core

Point avr-core to the Speechmatics STS service:

avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    - STS_URL=ws://avr-sts-speechmatics:6033
  ports:
    - 5001:5001
  networks:
    - avr

ℹ️ When using STS providers, AVR Core bypasses ASR + LLM + TTS and delegates speech handling entirely to the STS engine.

⸻

Audio Format

Speechmatics STS operates using the following format:
	•	Input: Linear PCM, 16-bit, 8kHz, mono
	•	Output: Linear PCM, 16-bit, 8kHz, mono

This format is fully compatible with Asterisk AudioSocket and AVR Core.

⸻

API Endpoint

POST /speech-to-speech-stream

Request
	•	Content-Type: audio/x-raw
	•	Audio: 16-bit PCM, 8kHz

Response
	•	Content-Type: text/event-stream
	•	Streamed PCM audio in real time

⸻

Performance Notes
	•	Ideal for real-time conversational agents
	•	Requires stable internet connectivity (cloud-based API)
	•	Simplifies architecture by removing intermediate speech components

⸻

Support & Community
	•	GitHub: https://github.com/agentvoiceresponse
	•	Discord: https://discord.gg/DFTU69Hg74
	•	Docker Hub: https://hub.docker.com/u/agentvoiceresponse
	•	Wiki: https://wiki.agentvoiceresponse.com/en/home

⸻

Support AVR

Agent Voice Response is free and open-source.
Support is optional and helps sustain development and community growth.

https://ko-fi.com/agentvoiceresponse

---

Se vuoi, nel prossimo step posso:
- allinearla **perfettamente alla sidebar della wiki**
- preparare **la versione con immagini**
- oppure scrivere **il messaggio di annuncio per Discord / LinkedIn**