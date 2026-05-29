---
title: Using xAI Grok Voice Agent STS with AVR
description: Configure avr-sts-xai for real-time Grok Voice Agent speech-to-speech with Agent Voice Response
published: true
date: 2026-05-29T21:45:00.000Z
tags: sts, xai, grok
editor: markdown
dateCreated: 2026-05-29T21:26:15.179Z
---

# Using xAI Grok Voice Agent STS with AVR

The **xAI Grok Voice Agent STS integration** connects Agent Voice Response to [xAI's real-time Voice Agent API](https://docs.x.ai/developers/model-capabilities/audio/voice-agent). AVR telephony audio is bridged over WebSocket to `wss://api.x.ai/v1/realtime`, with 8 kHz ↔ 24 kHz PCM resampling handled inside the connector.

This mode provides low-latency speech-to-speech conversations using Grok voice models, with support for AVR function tools and optional built-in xAI tools.

For AVR tool wiring, see [AVR Function Calls](https://wiki.agentvoiceresponse.com/en/avr-function-calls).

---

## Why Use xAI Grok Voice Agent STS?

- **Real-time speech-to-speech** — single WebSocket loop, no separate ASR/LLM/TTS pipeline
- **Grok voice models** — `grok-voice-latest`, `grok-voice-think-fast-1.0`, and versioned model names
- **Built-in voices** — `eve`, `ara`, `rex`, `sal`, `leo`
- **Barge-in** — user speech triggers `interruption` events to AVR core
- **Transcripts** — user and agent transcript events streamed to AVR
- **Optional xAI tools** — `web_search`, `x_search`, `file_search` via environment variables

---

## How to Obtain an xAI API Key

1. Sign up at [xAI Console](https://console.x.ai/).
2. Create or select a team/project.
3. Navigate to **API Keys**.
4. Generate a new key and copy it securely.
5. Set `XAI_API_KEY` in your connector environment.

Official API reference: https://docs.x.ai/developers/model-capabilities/audio/voice-agent

---

## Repository

- GitHub: https://github.com/agentvoiceresponse/avr-sts-xai
- Docker Hub: https://hub.docker.com/r/agentvoiceresponse/avr-sts-xai

---

## Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `XAI_API_KEY` | xAI API key | — | **Yes** |
| `PORT` | AVR-facing WebSocket port | `6041` | Optional |
| `XAI_MODEL` | Voice model query parameter | `grok-voice-latest` | Optional |
| `XAI_VOICE` | Agent voice (`eve`, `ara`, `rex`, `sal`, `leo`) | `eve` | Optional |
| `XAI_TURN_DETECTION` | Turn detection mode | `server_vad` | Optional |
| `XAI_TURN_DETECTION_THRESHOLD` | VAD threshold (when numeric) | — | Optional |
| `XAI_TURN_DETECTION_SILENCE_MS` | Silence duration for end-of-turn | — | Optional |
| `XAI_TURN_DETECTION_PREFIX_PADDING_MS` | Prefix padding for VAD | — | Optional |
| `XAI_TRANSCRIPTION_MODEL` | Input audio transcription model | `whisper-1` | Optional |
| `XAI_LANGUAGE` | Transcription language hint (e.g. `it`) | — | Optional |
| `XAI_CONNECT_TIMEOUT_MS` | Upstream xAI WebSocket connect timeout | `15000` | Optional |
| `XAI_BUILTIN_TOOLS` | Comma-separated: `web_search`, `x_search`, `file_search` | — | Optional |
| `XAI_COLLECTION_IDS` | Collection IDs for `file_search` | — | Optional |
| `AMI_URL` | AVR AMI service for transfer/hangup tools | `http://127.0.0.1:6006` | Optional |

### Instruction loading (priority order)

| Variable | Description |
|----------|-------------|
| `XAI_INSTRUCTIONS` | Inline system prompt (**highest priority**) |
| `XAI_URL_INSTRUCTIONS` | HTTP endpoint returning `{ "system": "..." }` (receives `X-AVR-UUID`) |
| `XAI_FILE_INSTRUCTIONS` | Local file path with plain-text instructions |
| *(fallback)* | Built-in default assistant prompt |

### Example `.env`

#### Required

```env
XAI_API_KEY=your_xai_api_key
```

#### Optional

```env
PORT=6041
XAI_MODEL=grok-voice-latest
XAI_VOICE=eve
XAI_INSTRUCTIONS="You are a helpful assistant that can answer questions and help with tasks."
XAI_TURN_DETECTION=server_vad
XAI_TRANSCRIPTION_MODEL=whisper-1
XAI_LANGUAGE=it
XAI_CONNECT_TIMEOUT_MS=15000
# XAI_BUILTIN_TOOLS=web_search,x_search
# XAI_COLLECTION_IDS=collection-id-1,collection-id-2
AMI_URL=http://avr-ami:6006
```

---

## Docker Compose Example

```yaml
avr-sts-xai:
  image: agentvoiceresponse/avr-sts-xai:1.0.0
  platform: linux/x86_64
  container_name: avr-sts-xai
  restart: always
  environment:
    - PORT=6041
    - XAI_API_KEY=${XAI_API_KEY}
    - XAI_MODEL=${XAI_MODEL:-grok-voice-latest}
    - XAI_VOICE=${XAI_VOICE:-eve}
    - XAI_INSTRUCTIONS=${XAI_INSTRUCTIONS:-You are a helpful assistant that can answer questions and help with tasks.}
    - XAI_TURN_DETECTION=${XAI_TURN_DETECTION:-server_vad}
    - AMI_URL=${AMI_URL:-http://avr-ami:6006}
  networks:
    - avr
```

A full stack example is available in [avr-infra `docker-compose-xai.yml`](https://github.com/agentvoiceresponse/avr-infra/blob/main/docker-compose-xai.yml).

---

## Integrating with AVR Core

Point **avr-core** at the xAI STS connector WebSocket endpoint:

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    - STS_URL=ws://avr-sts-xai:6041
  ports:
    - 5001:5001
  networks:
    - avr
```

`STS_URL` uses `ws://` with **no path suffix** (unlike HTTP-based ASR/LLM/TTS providers).

---

## AVR WebSocket Protocol

**Client → connector**

- `{"type":"init","uuid":"<session-uuid>"}`
- `{"type":"audio","audio":"<base64 pcm16 8kHz>"}`

**Connector → client**

- `{"type":"audio","audio":"<base64 pcm16 8kHz 20ms frame>"}`
- `{"type":"transcript","role":"user|agent","text":"..."}`
- `{"type":"interruption"}`
- `{"type":"error","message":"..."}`

See [AVR STS Integration Implementation](https://wiki.agentvoiceresponse.com/en/avr-sts-integration-implementation) for the full protocol reference.

---

## AVR Tools

The connector ships with built-in AVR tools:

- **`avr_transfer`** — transfer the call via AMI
- **`avr_hangup`** — hang up the call via AMI

Custom tools can be added under `tools/` and `avr_tools/`. Ensure `AMI_URL` points at your AVR AMI service when using transfer/hangup.

---

## Optional xAI Built-in Tools

Enable provider-side tools with a comma-separated list:

```env
XAI_BUILTIN_TOOLS=web_search,x_search,file_search
```

When using `file_search`, set collection IDs:

```env
XAI_COLLECTION_IDS=collection-id-1,collection-id-2
```

---

## Concurrency

Each AVR client connection uses dedicated audio resamplers and a dedicated xAI WebSocket session. Multiple concurrent calls are supported on a single connector instance.

---

## Troubleshooting

| Symptom | Check |
|---------|-------|
| Connector fails to start | `XAI_API_KEY` is set and valid |
| No audio from agent | `STS_URL=ws://avr-sts-xai:6041` on avr-core; connector reachable on the Docker network |
| Transfer/hangup not working | `AMI_URL` points to avr-ami; AMI credentials match Asterisk |
| Upstream timeout | Increase `XAI_CONNECT_TIMEOUT_MS` or verify outbound access to `api.x.ai` |
| Invalid instructions from URL/file | Payload must include `{ "system": "..." }` for URL mode; file must be readable plain text |

---

## Related Links

- [xAI Voice Agent API](https://docs.x.ai/developers/model-capabilities/audio/voice-agent)
- [AVR STS Integration Implementation](https://wiki.agentvoiceresponse.com/en/avr-sts-integration-implementation)
- [AVR Function Calls](https://wiki.agentvoiceresponse.com/en/avr-function-calls)
- [avr-infra docker-compose-xai.yml](https://github.com/agentvoiceresponse/avr-infra/blob/main/docker-compose-xai.yml)
