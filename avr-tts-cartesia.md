# Agent Voice Response with Cartesia TTS Integration

Overview
- This page documents the integration between AVR and Cartesia Text-to-Speech (TTS) API for real-time speech synthesis in telephony contexts.

## Features
- Real-time Text-to-Speech (TTS): Convert text to natural-sounding speech using Cartesia API.
- Streaming Audio: Audio is streamed back in real-time via Node.js streams for low-latency responses.

## Prerequisites
- Node.js and npm installed
- Cartesia API key

## Installation

1. Install dependencies (in the avr-tts-cartesia repository):
```bash
npm install
```

2. Create a .env file in the root with Cartesia credentials:
```env
CARTESIA_API_KEY=your_cartesia_api_key
CARTESIA_VOICE_ID=694f9389-aac1-45b6-b726-9d9369183238
CARTESIA_LANGUAGE=en
PORT=6009
```

## Usage

To start the service:
```bash
npm start
```

The service will listen on the port defined in the .env (default 6009).

### API Endpoint

POST /text-to-speech-stream

- Request body:
```json
{ "text": "Hello, how can I assist you today?", "voiceId": "optional_voice_id", "language": "en" }
```
- Response: Audio stream in audio/l16 (LINEAR16 PCM), mono, 8kHz.

### Optional Parameters
- `voiceId`: Cartesia voice ID (defaults to the `CARTESIA_VOICE_ID` env var)
- `language`: Language code (defaults to `en` or `CARTESIA_LANGUAGE` env var)

Example request:
```bash
curl -X POST http://localhost:6009/text-to-speech-stream \
  -H "Content-Type: application/json" \
  -d '{"text":"Hello, this is a real-time voice response!"}' \
  --output response.raw
```

## How It Works
- Sends the input text to Cartesia to synthesize speech using the configured voice.
- Streams synthesized audio back to the client in real-time.

## Error Handling
- 400 Bad Request if the text is missing.
- 500 Internal Server Error for Cartesia API issues.

## Docker Usage

- Run with docker:
```bash
docker run -p 6009:6009 -e CARTESIA_API_KEY=your_api_key agentvoiceresponse/avr-tts-cartesia
```

### Docker Compose Example

Here is a complete example of how to use avr-tts-cartesia in a docker-compose setup with avr-core:

```yaml
services:
  avr-core:
    image: agentvoiceresponse/avr-core
    platform: linux/x86_64
    container_name: avr-core
    restart: always
    environment:
      - PORT=5001
      - ASR_URL=http://avr-asr-deepgram:6010/speech-to-text-stream
      - LLM_URL=http://avr-llm-openai:6002/prompt-stream
      - TTS_URL=http://host.docker.internal:6009/text-to-speech-stream
      - INTERRUPT_LISTENING=true
      - SYSTEM_MESSAGE="Hello, how can I help you today?"
    ports:
      - "5001:5001"
    networks:
      - avr

  avr-tts-cartesia:
    image: agentvoiceresponse/avr-tts-cartesia
    platform: linux/x86_64
    container_name: avr-tts-cartesia
    restart: always
    environment:
      - CARTESIA_API_KEY=${CARTESIA_API_KEY}
      - CARTESIA_VOICE_ID=694f9389-aac1-45b6-b726-9d9369183238
      - CARTESIA_LANGUAGE=en
      - PORT=6009
    ports:
      - "6009:6009"
    networks:
      - avr

networks:
  avr:
    driver: bridge
```

This configuration:
- avr-core runs on port 5001 (default)
- Uses ASR_URL and LLM_URL to connect to other AVR services
- Uses `host.docker.internal` for TTS_URL so avr-core can reach the TTS service running on the host
- Both services are on the same Docker network for internal communication

## Architecture
- See diagram: avr-tts-cartesia-diagram.drawio (or inline ASCII diagram if needed)

## Environment Variables (Complete Snippet)
```env
CARTESIA_API_KEY=your_cartesia_api_key
CARTESIA_VOICE_ID=694f9389-aac1-45b6-b726-9d9369183238
CARTESIA_LANGUAGE=en
PORT=6009
```

## References
- Cartesia API docs: https://docs.cartesia.ai
- AVR project repo: https://github.com/agentvoiceresponse/avr-tts-cartesia

### End of page
