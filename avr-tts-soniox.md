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

Wiki path: `/avr-tts-soniox`

## Why use Soniox with AVR?

- Real-time optimized TTS for conversational voice agents.
- AVR-compatible HTTP interface and streaming behavior.
- Configurable model, voice, language, format, sample rate, and bitrate.

## Runtime behavior

`avr-tts-soniox` is a streaming bridge between AVR and Soniox real-time TTS:

1. AVR sends a `POST` request with text (and optional voice/model/audio overrides).
2. The connector forwards the request upstream to Soniox.
3. Audio bytes are streamed back chunk-by-chunk to AVR with backpressure handling.
4. If the AVR client disconnects, the connector aborts the upstream request and releases resources.
5. Upstream trailer-level errors are detected and emitted as connector stream-error events.

This keeps streaming deterministic and avoids long-lived hanging sockets in call flows.

## Endpoint contract

`POST /text-to-speech-stream`

Example request:

```json
{
  "text": "Hello from Soniox Text to Speech",
  "voice": "Adrian",
  "language": "en",
  "audio_format": "pcm_s16le",
  "sample_rate": 8000,
  "model": "tts-rt-v1"
}
```

Example local call:

```bash
curl -X POST http://localhost:6011/text-to-speech-stream \
  -H "Content-Type: application/json" \
  -H "x-uuid: call-1234" \
  -d '{"text":"Welcome to Agent Voice Response."}' \
  --output response.pcm
```

### Request fields

| Field | Required | Description |
|----------|----------|-------------|
| `text` | Yes | Text to synthesize |
| `model` | No | Soniox model override |
| `voice` | No | Voice override |
| `language` | No | Language override |
| `audio_format` / `audioFormat` | No | Output format override |
| `sample_rate` / `sampleRate` | No | Sample-rate override |
| `bitrate` | No | Optional bitrate for compressed formats |

Parameter precedence:

1. Request body fields
2. `SONIOX_TTS_*` environment defaults
3. Connector built-in defaults

## Environment variables and defaults

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

## Content type and audio expectations

- For PCM formats (`pcm_*`), the connector returns `Content-Type: audio/l16`.
- For non-PCM formats, the connector returns `application/octet-stream`.
- The telephony default path is `pcm_s16le` at `8000` Hz for AVR compatibility.

## Error handling and failure modes

The connector is designed to fail predictably for production call flows:

- `400` when `text` is missing or invalid.
- `500` when `SONIOX_API_KEY` is missing.
- `504` when Soniox upstream exceeds `SONIOX_TTS_TIMEOUT_MS`.
- Upstream non-2xx responses are propagated with structured JSON payload:
  - `error: soniox_tts_request_failed`
  - `details` and optional upstream `code`
- Mid-stream client disconnects stop upstream work and close the response cleanly.

### Troubleshooting quick checks

1. **No audio on call**
   - Verify `TTS_URL` points to `/text-to-speech-stream`.
   - Confirm output format is PCM (`pcm_s16le`) with `8000` Hz for telephony.
2. **Intermittent stalls**
   - Check `SONIOX_TTS_TIMEOUT_MS` and upstream latency.
   - Validate network path between connector and Soniox endpoint.
3. **Unexpected stream close**
   - Inspect trailer errors (`x-tts-error-code`, `x-tts-error-message`) in logs/events.
   - Correlate requests using `x-uuid` header.
4. **5xx from connector**
   - Confirm `SONIOX_API_KEY` is configured and valid.
   - Verify Soniox endpoint URL and model/voice parameters.

## Observability signals

For production monitoring and incident triage:

- Correlation ID support via `x-uuid` request header.
- Stream-error event emission when Soniox returns trailer error metadata.
- Distinct timeout error shape: `soniox_tts_upstream_timeout`.
- Connector startup log includes listening port (`[avr-tts-soniox] listening on ...`).

## Validation evidence

Implementation validation includes automated coverage for key real-time failure modes:

- Stream with upstream trailer errors is detected and reported.
- Non-zero streaming bytes complete successfully when no trailer error exists.
- Deterministic `504` behavior for upstream timeout path.
- Listener cleanup in repeated backpressure waits (leak prevention for long-lived sessions).

Reference tests: [`test/tts-stream.test.js`](https://github.com/agentvoiceresponse/avr-tts-soniox/blob/main/test/tts-stream.test.js)

## AVR infra compose wiring

Use `docker-compose-soniox.yml` from `avr-infra` with:

- `TTS_URL=http://avr-tts-soniox:6011/text-to-speech-stream`
- service `avr-tts-soniox` image `agentvoiceresponse/avr-tts-soniox`

Infra reference: [docker-compose-soniox.yml](https://github.com/agentvoiceresponse/avr-infra/blob/main/docker-compose-soniox.yml)

## Traceability

- Implementation task: [AVR-36](/AVR/issues/AVR-36)
- Documentation task: [AVR-39](/AVR/issues/AVR-39)
- Parent delivery: [AVR-35](/AVR/issues/AVR-35)

## References

- Soniox TTS getting started: [https://soniox.com/docs/tts/get-started](https://soniox.com/docs/tts/get-started)
- Soniox REST generate speech: [https://soniox.com/docs/tts/rest-api/generate-speech](https://soniox.com/docs/tts/rest-api/generate-speech)
- Source repository: [https://github.com/agentvoiceresponse/avr-tts-soniox](https://github.com/agentvoiceresponse/avr-tts-soniox)
