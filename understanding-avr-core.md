---
title: Understanding AVR Core
description: 
published: true
date: 2025-09-30T11:10:30.086Z
tags: asr, tts, asterisk, avr-core, architecture, integration, voip, llm, sts
editor: markdown
dateCreated: 2025-09-30T11:07:10.215Z
---

# AVR Core

The **Agent Voice Response (AVR) Core** is the real-time orchestrator that connects your PBX (Asterisk via **AudioSocket**) with AI services. It replaces rigid IVRs with **conversational voicebots** by streaming audio in, routing it to **ASR / LLM / TTS** (or a single **STS** provider), and streaming synthesized audio back to callers with minimal latency.

## Overview

AVR Core manages live call media and conversation state:

1. **ASR (Automatic Speech Recognition)** — transcribes caller audio to text.  
2. **LLM (Large Language Model)** — reasons on the transcript + context to produce a reply.  
3. **TTS (Text-to-Speech)** — renders the reply as audio back to the caller.  
4. **STS (Speech-to-Speech)** — optional *single-hop* pipeline: speech in → speech out (bypasses ASR/LLM/TTS).

AVR Core is **provider-agnostic**. It talks to modules over **streaming HTTP** (and **WebSocket** for some STS providers), so you can mix-and-match vendors or build your own adapters.

## Call Flows

### ASR → LLM → TTS (text-mediated)
- Asterisk sends RTP audio to AVR Core via **AudioSocket**.
- Core streams audio chunks to **ASR** (`ASR_URL=/speech-to-text-stream`).
- Final transcript is sent to **LLM** (`LLM_URL=/prompt-stream`).
- LLM reply text is sent to **TTS** (`TTS_URL=/text-to-speech-stream`).
- Core streams the synthesized audio back to Asterisk.

### STS (speech-mediated)
- Asterisk streams audio to Core via **AudioSocket**.
- Core streams speech to **STS** (`STS_URL=WS provider-dependent).
- STS returns synthesized speech; Core streams it back to Asterisk.

> Use **STS** for the lowest latency; use **ASR+LLM+TTS** for maximal control, tools/function-calling, and explainability.

## Features

- **Plug-and-play architecture** — swap ASR/LLM/TTS/STS via URLs.
- **True streaming** end-to-end (AudioSocket ↔ HTTP/WS streams).
- **Multi-codec support** — automatic detection of **μ-law (ulaw)**, **A-law (alaw)**, **Linear PCM (slin16)**.
- **Webhooks** — call lifecycle, interruption, transcription, DTMF events.
- **Ambient noise** — optional background sound mixing for natural feel.
- **VAD (Voice Activity Detection)** — fast turn-taking, barge-in handling.
- **Scalable & modular** — run each component independently (containers).

For ready-made adapters, see the organization repositories:  
**Integrations:** https://github.com/orgs/agentvoiceresponse/repositories

---

## Installation

### 1) Get the infrastructure
Clone **avr-infra** for Docker Compose templates:
```bash
git clone https://github.com/agentvoiceresponse/avr-infra
cd avr-infra
```

### 2) Configure environment
Set the relevant variables in your `.env` (examples below). Core will use either **ASR+LLM+TTS** *or* **STS** depending on what you set.

#### Core (text-mediated)
```env
ASR_URL=http://avr-asr-*:6010/speech-to-text-stream
LLM_URL=http://avr-llm-*:6002/prompt-stream
TTS_URL=http://avr-tts-*:6012/text-to-speech-stream
```

#### Core (speech-mediated)
```env
STS_URL=http://avr-sts-*:6033/speech-to-speech-stream
# When STS_URL is set, comment out ASR_URL / LLM_URL / TTS_URL.
```

Optional:
```env
WEBHOOK_URL=https://yourapp.example.com/avr-webhook
WEBHOOK_SECRET=supersecret
WEBHOOK_TIMEOUT=3000
WEBHOOK_RETRY=0

AMBIENT_NOISE_FILE=ambient_sounds/office_background.raw
AMBIENT_NOISE_LEVEL=0.50  # 0.0–1.0
```

### 3) Example docker-compose (Core only)
```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    # Choose either ASR+LLM+TTS ...
    # - ASR_URL=http://avr-asr-deepgram:6010/speech-to-text-stream
    # - LLM_URL=http://avr-llm-openai:6002/prompt-stream
    # - TTS_URL=http://avr-tts-kokoro:6012/text-to-speech-stream
    # ... or STS
    # - STS_URL=http://avr-sts-openai:6030/speech-to-speech-stream
    # Optional webhooks & ambient noise
    - WEBHOOK_URL=${WEBHOOK_URL}
    - WEBHOOK_SECRET=${WEBHOOK_SECRET}
    - AMBIENT_NOISE_FILE=ambient_sounds/office_background.raw
    - AMBIENT_NOISE_LEVEL=0.10
  volumes:
    - ./ambient_sounds:/usr/src/app/ambient_sounds
  ports:
    - 5001:5001
  networks:
    - avr
```

---

## Asterisk Integration

AVR Core connects over **AudioSocket**. You can integrate via the **AudioSocket() application** or the **Dial(AudioSocket/…)** channel interface.

### Option A — `AudioSocket()` (simple, auto-transcoding to slin16)
```asterisk
[avr]
exten => 5001,1,Answer()
 same => n,Ringing()
 same => n,Wait(1)
 same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
 same => n,AudioSocket(${UUID},IP_AVR_CORE:5001)
 same => n,Hangup()
```
**Pros:** Asterisk will transcode incoming audio to **slin16** (what Core expects), minimizing codec issues.

### Option B — `Dial(AudioSocket/...)` (scalable, but codec-sensitive)
```asterisk
[avr]
exten => 5001,1,Answer()
 same => n,Ringing()
 same => n,Wait(1)
 same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
 same => n,Dial(AudioSocket/IP_AVR_CORE:5001/${UUID})
 same => n,Hangup()
```
**Note:** With `Dial(AudioSocket/...)` Asterisk passes the **native endpoint codec** (e.g., Opus if your softphone uses Opus).  
If Core receives an unsupported codec, the call may drop immediately. Restrict endpoint codecs in `pjsip.conf`:
```ini
[endpoint-template](!)
type=endpoint
disallow=all
allow=alaw        ; or 'ulaw' or 'slin16'
```

**Debug tip**
```bash
asterisk -rx "core show channel AudioSocket/IP:PORT-XXXX"
```
Check `ReadFormat` is `slin` (or a supported G.711). If you see `opus`, restrict codecs as above **or** prefer `AudioSocket()`.

---

## Webhook Integration

AVR Core can POST events to your app for monitoring and automation.

**Events**
- `call_started` — a new call session begins  
- `call_ended` — the call ends  
- `interruption` — user barges in / interrupts TTS  
- `transcription` — transcript item available (also in STS mode if exposed)  
- `dtmf_digit` — DTMF received

**Payload**
```json
{
  "uuid": "call-session-uuid",
  "type": "event_type",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "payload": { /* event-specific */ }
}
```

**Config**
```env
WEBHOOK_URL=https://yourapp.example.com/hooks/avr
WEBHOOK_SECRET=optional-shared-secret
WEBHOOK_TIMEOUT=3000
WEBHOOK_RETRY=0
```

## Audio Codec Support

AVR Core **auto-detects** the inbound codec (μ-law, A-law, slin16) from the first packets, logs the detection, decodes to **internal 8 kHz / 16-bit mono PCM**, and re-encodes for outbound streaming.  

**Best practices**
- Keep codecs consistent end-to-end (prefer **alaw/ulaw/slin16**).  
- Avoid Opus on the AudioSocket leg unless Asterisk transcodes to slin16.  
- Monitor logs: you’ll see `Audio codec detected: ALAW/ULAW/SLIN`.

## Ambient Noise & VAD

- **Ambient Noise**  
  ```env
  AMBIENT_NOISE_FILE=ambient_sounds/office_background.raw
  AMBIENT_NOISE_LEVEL=0.10
  ```
  Mount the folder into the Core container and point to a **raw, mono, 8 kHz, 16-bit PCM** file.

- **VAD (Voice Activity Detection)**  
  Core uses VAD to segment speech, reduce latency, and support **barge-in**. Defaults are tuned for responsiveness; adjust only if you have specific needs.

---

## Environment Variables (Summary)

| Variable              | Purpose                                   | Example / Notes                                           |
|-----------------------|-------------------------------------------|-----------------------------------------------------------|
| `PORT`                | Core listen port                          | `5001`                                                    |
| `ASR_URL`             | ASR streaming endpoint                    | `http://avr-asr-*:6010/speech-to-text-stream`            |
| `LLM_URL`             | LLM streaming endpoint                    | `http://avr-llm-*:6002/prompt-stream`                    |
| `TTS_URL`             | TTS streaming endpoint                    | `http://avr-tts-*:6012/text-to-speech-stream`            |
| `STS_URL`             | STS streaming endpoint (WS)          | `ws://avr-sts-*:6033`          |
| `WEBHOOK_URL`         | Webhook receiver                          | `https://yourapp/hook`                                   |
| `WEBHOOK_SECRET`      | Signature secret (header: `X-AVR-WEBHOOK-SECRET`) | Optional                                           |
| `WEBHOOK_TIMEOUT`     | Webhook request timeout (ms)              | `3000`                                                    |
| `WEBHOOK_RETRY`       | Retries for failed webhooks               | `0`                                                       |
| `AMBIENT_NOISE_FILE`  | Ambient PCM file (8 kHz, mono, 16-bit)    | `ambient_sounds/office_background.raw`                    |
| `AMBIENT_NOISE_LEVEL` | Ambient volume (0.0–1.0)                  | `0.10`                                                    |

---

## Performance & Scaling

- **Latency:** Use STS for ultra-low latency; otherwise keep ASR/LLM/TTS local to minimize network hops.  
- **CPU:** Transcoding (G.711 ↔ slin) costs CPU; prefer consistent codecs.  
- **Horizontal scale:** Run multiple Core instances behind a TCP load balancer; scale ASR/LLM/TTS/STS independently.  
- **Monitoring:** Tail Core logs for codec detection and stream timing; add webhooks to feed analytics/QA.

---

## Troubleshooting

- **Immediate hang-up:** Likely codec mismatch (e.g., Opus). Use `AudioSocket()` or restrict endpoint codecs to `alaw/ulaw/slin16`.  
- **“No audio received / unsupported format”:** Verify Asterisk channel `ReadFormat`.  
- **Long delays:** Check ASR/LLM/TTS response times; consider smaller models or local providers.  
- **Webhook timeouts:** Increase `WEBHOOK_TIMEOUT` or ensure your endpoint responds < 3s.
