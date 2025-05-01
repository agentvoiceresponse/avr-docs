# TTS (Text-to-Speech) Integrations

## Overview

AVR supports multiple TTS providers for converting text to natural-sounding speech. Each provider offers different voice options, languages, and quality levels.

## Available Providers

### 1. Deepgram (`avr-tts-deepgram`)

**Features:**
- High-quality voices
- Multiple language support
- Real-time streaming
- Commercial API

**Configuration:**
```bash
# Environment Variables
TTS_URL=http://localhost:6003/text-to-speech-stream
DEEPGRAM_API_KEY=your_api_key
VOICE_ID=your_voice_id
```

**Docker Compose:**
```yaml
services:
  avr-tts-deepgram:
    image: agentvoiceresponse/avr-tts-deepgram
    environment:
      - PORT=6003
      - DEEPGRAM_API_KEY=${DEEPGRAM_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

### 2. ElevenLabs (`avr-tts-elevenlabs`)

**Features:**
- Highly realistic voices
- Voice cloning
- Emotional range
- Commercial API

**Configuration:**
```bash
# Environment Variables
TTS_URL=http://localhost:6003/text-to-speech-stream
ELEVENLABS_API_KEY=your_api_key
VOICE_ID=your_voice_id
```

**Docker Compose:**
```yaml
services:
  avr-tts-elevenlabs:
    image: agentvoiceresponse/avr-tts-elevenlabs
    environment:
      - PORT=6003
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

### 3. Google Cloud TTS (`avr-tts-google-cloud-tts`)

**Features:**
- Wide range of voices
- Multiple languages
- Neural voices
- Cloud-based service

**Configuration:**
```bash
# Environment Variables
TTS_URL=http://localhost:6003/text-to-speech-stream
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
VOICE_NAME=your_voice_name
```

**Docker Compose:**
```yaml
services:
  avr-tts-google:
    image: agentvoiceresponse/avr-tts-google-cloud-tts
    environment:
      - PORT=6003
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json
      - VOICE_NAME=${VOICE_NAME}
    volumes:
      - ./credentials.json:/app/credentials.json
```

### 4. Kokoro (`avr-tts-kokoro`)

**Features:**
- Japanese language focus
- High-quality voices
- Emotional expression
- Commercial API

**Configuration:**
```bash
# Environment Variables
TTS_URL=http://localhost:6003/text-to-speech-stream
KOKORO_API_KEY=your_api_key
VOICE_ID=your_voice_id
```

**Docker Compose:**
```yaml
services:
  avr-tts-kokoro:
    image: agentvoiceresponse/avr-tts-kokoro
    environment:
      - PORT=6003
      - KOKORO_API_KEY=${KOKORO_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

## Integration Guide

### 1. Choose a Provider

Select a TTS provider based on your requirements:
- For general use: Deepgram or Google Cloud TTS
- For highly realistic voices: ElevenLabs
- For Japanese language: Kokoro

### 2. Configure Environment

1. Set up the required environment variables
2. Configure API keys
3. Select appropriate voice parameters

### 3. Start the Service

```bash
# Using Docker Compose
docker-compose -f docker-compose-{provider}.yml up -d
```

### 4. Test the Integration

```bash
# Example curl command to test the service
curl -X POST http://localhost:6003/text-to-speech-stream \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello, how can I help you?", "voice_id": "your_voice_id"}'
```

## Best Practices

1. **Voice Selection**
   - Choose appropriate voice for your audience
   - Consider language and accent requirements
   - Test voice quality in different scenarios

2. **Performance**
   - Monitor audio streaming quality
   - Implement proper buffering
   - Handle network latency

3. **Cost Management**
   - Monitor API usage
   - Implement caching for common phrases
   - Use appropriate quality settings

## Troubleshooting

### Common Issues

1. **Audio Quality Issues**
   - Check audio encoding settings
   - Verify voice parameters
   - Test different sample rates

2. **Latency Problems**
   - Monitor network connectivity
   - Check service response times
   - Review buffer settings

3. **API Errors**
   - Verify API keys
   - Check service health
   - Review error logs

## Support

For issues specific to TTS integrations:
- Check provider documentation
- Review service logs
- Contact support at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com) 