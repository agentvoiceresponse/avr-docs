---
title: speech-to-text
description: 
published: true
date: 2025-05-01T19:28:58.964Z
tags: 
editor: markdown
dateCreated: 2025-05-01T19:12:26.284Z
---

# STT (Speech To Text) Integrations

## Overview

AVR provides a middleware container (`avr-asr-to-stt`) that bridges between `avr-core` and various STT providers. This allows integration with any speech-to-text service that doesn't support direct ASR integration.

## Architecture

```
[avr-core] <-> [avr-asr-to-stt] <-> [avr-stt-{provider}]
```

The `avr-asr-to-stt` container:
1. Receives audio streams from `avr-core`
2. Processes the audio using Silero VAD
3. Forwards the processed audio to the STT provider
4. Returns the transcription to `avr-core`

## Available STT Providers

### 1. Gemini (`avr-stt-gemini`)

**Features:**
- Google's Gemini model for speech recognition
- High accuracy
- Multiple language support
- Cloud-based service

**Configuration:**
```yaml
# docker-compose.yml
services:
  avr-asr-to-stt:
    image: agentvoiceresponse/avr-asr-to-stt
    environment:
      - PORT=6001
      - MODEL_PATH=/app/silero_vad.onnx
      - STT_URL=http://avr-stt-gemini:6002/text-to-speech
    volumes:
      - ./silero_vad.onnx:/app/silero_vad.onnx

  avr-stt-gemini:
    image: agentvoiceresponse/avr-stt-gemini
    environment:
      - PORT=6002
      - GEMINI_API_KEY=${GEMINI_API_KEY}
```

**Implementation Example:**
```javascript
// Processing audio with Gemini
const gemini = new Gemini({
  credentials: process.env.GEMINI_API_KEY,
  languageCode: 'en-US'
});

async function processAudio(audioBuffer) {
  const transcription = await gemini.transcribe(audioBuffer);
  return transcription.text;
}
```

### 2. ElevenLabs (`avr-stt-elevenlabs`)

**Features:**
- High accuracy speech recognition
- Real-time processing
- Multiple language support
- Commercial API

**Configuration:**
```yaml
# docker-compose.yml
services:
  avr-asr-to-stt:
    image: agentvoiceresponse/avr-asr-to-stt
    environment:
      - PORT=6001
      - MODEL_PATH=/app/silero_vad.onnx
      - STT_URL=http://avr-stt-elevenlabs:6002/text-to-speech

  avr-stt-elevenlabs:
    image: agentvoiceresponse/avr-stt-elevenlabs
    environment:
      - PORT=6002
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}
      - LANGUAGE=en
```

**Implementation Example:**
```javascript
// Processing audio with ElevenLabs
const elevenlabs = new ElevenLabs({
  apiKey: process.env.ELEVENLABS_API_KEY,
  language: 'en'
});

async function processAudio(audioBuffer) {
  const transcription = await elevenlabs.transcribe(audioBuffer);
  return transcription.text;
}
```

## Integration Guide

### 1. Setup avr-asr-to-stt

1. **Install Silero VAD Model**
   ```bash
   wget https://github.com/snakers4/silero-vad/raw/master/files/silero_vad.onnx
   ```

2. **Configure Environment**
   ```bash
   # .env
   MODEL_PATH=/path/to/silero_vad.onnx
   STT_URL=http://avr-stt-{provider}:6002/text-to-speech
   ```

3. **Start the Container**
   ```bash
   docker-compose -f docker-compose-asr-to-stt.yml up -d
   ```

### 2. Setup STT Provider

1. **Choose a Provider**
   - Gemini for Google ecosystem integration
   - ElevenLabs for high-quality commercial service

2. **Configure Provider**
   - Set up API keys
   - Configure language settings
   - Set up any required credentials

3. **Start the Provider**
   ```bash
   docker-compose -f docker-compose-stt-{provider}.yml up -d
   ```

### 3. Test the Integration

```bash
# Test audio streaming
curl -X POST http://localhost:6001/speech-to-text-stream \
  -H "Content-Type: audio/wav" \
  --data-binary @test.wav
```

## Best Practices

1. **Audio Processing**
   - Use appropriate sample rates (16kHz recommended)
   - Implement noise reduction
   - Handle audio buffering properly

2. **Performance**
   - Monitor latency between components
   - Implement proper error handling
   - Use appropriate buffer sizes

3. **Error Handling**
   - Handle network issues
   - Implement retry mechanisms
   - Log errors appropriately

## Troubleshooting

### Common Issues

1. **Audio Quality**
   - Check sample rate settings
   - Verify audio format
   - Test with different microphones

2. **Connection Issues**
   - Verify network connectivity
   - Check container health
   - Review service logs

3. **Recognition Problems**
   - Check language settings
   - Verify API credentials
   - Test with different audio samples

## Support

For issues with STT integrations:
- Check provider documentation
- Review service logs
- Contact support at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com)

## Related Documentation

- [ASR Integrations](Automatic Speech Recognition.md) - For direct ASR integrations
- [Performance Optimization](../Advanced/Performance Optimization Guide.md) - For optimizing STT performance
- [Security Best Practices](../Advanced/Security Best Practices.md) - For securing STT integrations 