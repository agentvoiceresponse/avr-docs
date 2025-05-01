---
title: asr
description: 
published: true
date: 2025-05-01T18:05:44.711Z
tags: 
editor: markdown
dateCreated: 2025-05-01T18:05:40.266Z
---

# ASR (Automatic Speech Recognition) Integrations

## Overview

AVR supports multiple ASR providers for converting speech to text. Each provider has its own strengths in terms of accuracy, language support, and features.

## Available Providers

### 1. Silero VAD (`avr-asr-to-stt`)

**Features:**
- Voice Activity Detection (VAD)
- Real-time streaming
- Low latency
- Open-source

**Configuration:**
```bash
# Environment Variables
ASR_URL=http://localhost:6001/speech-to-text-stream
SILERO_MODEL_PATH=/path/to/silero_vad.onnx
```

**Docker Compose:**
```yaml
services:
  avr-asr-silero:
    image: agentvoiceresponse/avr-asr-to-stt
    environment:
      - PORT=6001
      - MODEL_PATH=/app/silero_vad.onnx
    volumes:
      - ./silero_vad.onnx:/app/silero_vad.onnx
```

### 2. ElevenLabs (`avr-stt-elevenlabs`)

**Features:**
- High accuracy
- Multiple language support
- Real-time streaming
- Commercial API

**Configuration:**
```bash
# Environment Variables
ASR_URL=http://localhost:6001/speech-to-text-stream
ELEVENLABS_API_KEY=your_api_key
```

**Docker Compose:**
```yaml
services:
  avr-asr-elevenlabs:
    image: agentvoiceresponse/avr-stt-elevenlabs
    environment:
      - PORT=6001
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}
```

### 3. Gemini (`avr-stt-gemini`)

**Features:**
- Google's speech recognition
- High accuracy
- Multiple language support
- Cloud-based service

**Configuration:**
```bash
# Environment Variables
ASR_URL=http://localhost:6001/speech-to-text-stream
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

**Docker Compose:**
```yaml
services:
  avr-asr-gemini:
    image: agentvoiceresponse/avr-stt-gemini
    environment:
      - PORT=6001
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json
    volumes:
      - ./credentials.json:/app/credentials.json
```

## Integration Guide

### 1. Choose a Provider

Select an ASR provider based on your requirements:
- For open-source solution: Silero VAD
- For commercial use with high accuracy: ElevenLabs
- For Google ecosystem integration: Gemini

### 2. Configure Environment

1. Set up the required environment variables
2. Configure API keys or credentials
3. Set up any required model files

### 3. Start the Service

```bash
# Using Docker Compose
docker-compose -f docker-compose-{provider}.yml up -d
```

### 4. Test the Integration

```bash
# Example curl command to test the service
curl -X POST http://localhost:6001/speech-to-text-stream \
  -H "Content-Type: audio/wav" \
  --data-binary @test.wav
```

## Best Practices

1. **Audio Format**
   - Use appropriate sample rates (16kHz recommended)
   - Ensure proper audio encoding
   - Consider noise reduction for better accuracy

2. **Performance**
   - Monitor latency
   - Adjust buffer sizes
   - Implement proper error handling

3. **Cost Optimization**
   - Implement caching for repeated phrases
   - Use appropriate quality settings
   - Monitor API usage

## Troubleshooting

### Common Issues

1. **High Latency**
   - Check network connectivity
   - Verify audio buffer settings
   - Monitor service logs

2. **Poor Accuracy**
   - Verify audio quality
   - Check language settings
   - Ensure proper microphone setup

3. **Connection Issues**
   - Verify API keys
   - Check service health
   - Review network configuration

## Support

For issues specific to ASR integrations:
- Check provider documentation
- Review service logs
- Contact support at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com) 