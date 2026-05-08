---
title: Soniox TTS
description:
published: true
date: 2026-05-07T19:30:00.000Z
tags:
editor: markdown
dateCreated: 2026-05-07T19:30:00.000Z
---

# Soniox TTS Integration

**Soniox** provides low-latency text-to-speech synthesis with streaming support. The [`avr-tts-soniox`](https://github.com/agentvoiceresponse/avr-tts-soniox) connector keeps AVR compatibility by exposing the standard `POST /text-to-speech-stream` route and streaming audio chunks directly to AVR Core.

## Why use Soniox with AVR?

- Real-time optimized TTS for conversational voice agents.
- AVR-compatible HTTP interface and streaming behavior.
- Configurable model, voice, language, format, sample rate, and bitrate.

## Endpoint

`POST /text-to-speech-stream`

Example request:

```json
{
  "text": "Hello from Soniox Text to Speech",
  "voice": "Adrian",
  "language": "en",
  "audio_format": "pcm_s16le",
  "sample_rate": 8000
}
```

Example local call:

```bash
curl -X POST http://localhost:6011/text-to-speech-stream \
  -H "Content-Type: application/json" \
  -d '{"text":"Welcome to Agent Voice Response."}' \
  --output response.pcm
```

## Environment variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `SONIOX_API_KEY` | Yes | - | Soniox API key used for Bearer auth |
| `SONIOX_TTS_URL` | No | `https://tts-rt.soniox.com/tts` | Soniox REST TTS endpoint |
| `SONIOX_TTS_MODEL` | No | `tts-rt-v1` | Soniox model |
| `SONIOX_TTS_LANGUAGE` | No | `en` | Default language |
| `SONIOX_TTS_VOICE` | No | `Adrian` | Default voice |
| `SONIOX_TTS_AUDIO_FORMAT` | No | `pcm_s16le` | Output format |
| `SONIOX_TTS_SAMPLE_RATE` | No | `8000` | Default sample rate |
| `SONIOX_TTS_BITRATE` | No | - | Optional bitrate for compressed formats |
| `SONIOX_TTS_TIMEOUT_MS` | No | `30000` | Upstream request timeout (ms) |
| `PORT` | No | `6011` | Connector HTTP port |

## AVR Infra compose

Use `docker-compose-soniox.yml` from `avr-infra` with:

- `TTS_URL=http://avr-tts-soniox:6011/text-to-speech-stream`
- service `avr-tts-soniox` image `agentvoiceresponse/avr-tts-soniox`

## References

- Soniox TTS getting started: [https://soniox.com/docs/tts/get-started](https://soniox.com/docs/tts/get-started)
- Soniox REST generate speech: [https://soniox.com/docs/tts/rest-api/generate-speech](https://soniox.com/docs/tts/rest-api/generate-speech)
