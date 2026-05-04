---
title: Cartesia TTS
description:
published: true
date: 2026-05-05T12:00:00.000Z
tags:
editor: markdown
dateCreated: 2026-05-05T12:00:00.000Z
---

# Cartesia TTS Integration

**Cartesia** provides a cloud text-to-speech API tuned for low-latency streaming. The [`avr-tts-cartesia`](https://github.com/agentvoiceresponse/avr-tts-cartesia) microservice sits between AVR Core and Cartesia: it accepts text from AVR, calls Cartesia to synthesize speech, and streams **8 kHz mono LINEAR16 PCM** audio back for telephony use (for example via Asterisk AudioSocket).

---

## Why use Cartesia with AVR?

- **Real-time streaming** — Audio is streamed to the client as it is generated.
- **Natural voices** — High-quality neural voices via the Cartesia API.
- **Simple HTTP surface** — Same `POST /text-to-speech-stream` pattern as other AVR TTS adapters.

---

## Prerequisites

- **Node.js** and **npm** (for local runs from the repository).
- A **Cartesia API key** from [Cartesia](https://cartesia.ai).

---

## Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/agentvoiceresponse/avr-tts-cartesia.git
   cd avr-tts-cartesia
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Create a `.env` file in the project root:

   ```env
   CARTESIA_API_KEY=your_cartesia_api_key
   CARTESIA_VOICE_ID=optional_default_voice_id
   CARTESIA_LANGUAGE=en
   PORT=6009
   ```

---

## Usage

Start the service:

```bash
npm start
```

The process listens on `PORT` from `.env` (default **6009**).

### API: `POST /text-to-speech-stream`

Accepts JSON with the text to synthesize. The response is streamed as `audio/l16`: **mono, 8 kHz, 16-bit linear PCM**.

**Request body**

```json
{
  "text": "Hello, how can I assist you today?",
  "voiceId": "optional_voice_id",
  "language": "en"
}
```

| Field | Description |
|--------|---------------|
| `text` | **Required.** Text to speak. |
| `voiceId` | Optional Cartesia voice ID. If omitted, `CARTESIA_VOICE_ID` from the environment is used. |
| `language` | Optional language code. If omitted, the service uses `CARTESIA_LANGUAGE` or a sensible default. |

**Example**

```bash
curl -X POST http://localhost:6009/text-to-speech-stream \
  -H "Content-Type: application/json" \
  -d '{"text":"Hello, this is a real-time voice response!"}' \
  --output response.raw
```

`response.raw` contains PCM suitable for AVR’s telephony pipeline.

---

## How it works

1. AVR Core sends text to the `avr-tts-cartesia` service (same integration pattern as other TTS microservices).
2. The service calls the Cartesia API to synthesize audio for the configured voice and language.
3. Synthesized audio is streamed back in real time as **8 kHz LINEAR16** for playback on the call.

---

## Error handling

- **400 Bad Request** — Missing or invalid `text` in the JSON body.
- **500 Internal Server Error** — Cartesia API or upstream errors.

---

## Docker

Run the published image:

```bash
docker run -p 6009:6009 -e CARTESIA_API_KEY=your_api_key agentvoiceresponse/avr-tts-cartesia
```

From the repository you can also use Compose (see the project’s `docker-compose` files).

---

## Environment variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `CARTESIA_API_KEY` | Cartesia API key | Yes | — |
| `CARTESIA_VOICE_ID` | Default voice ID when the request does not specify `voiceId` | No | Service default |
| `CARTESIA_LANGUAGE` | Default language code | No | `en` (or service default) |
| `PORT` | HTTP port for the microservice | No | `6009` |

---

## References

- Cartesia API documentation: [https://docs.cartesia.ai](https://docs.cartesia.ai)
- Source repository: [https://github.com/agentvoiceresponse/avr-tts-cartesia](https://github.com/agentvoiceresponse/avr-tts-cartesia)
- Community: [AVR on Discord](https://discord.gg/DFTU69Hg74)
