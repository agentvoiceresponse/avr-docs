---
title: automatic-speech-recognition
description: 
published: true
date: 2025-05-01T19:25:43.976Z
tags: 
editor: markdown
dateCreated: 2025-05-01T19:12:09.066Z
---

# ASR (Automatic Speech Recognition) Integrations

## Overview

AVR supports direct ASR integrations with providers that offer real-time speech recognition capabilities. These integrations provide low-latency, high-accuracy speech-to-text conversion.

## Available Providers

### 1. Deepgram (`avr-asr-deepgram`)

**Features:**
- High accuracy speech recognition
- Real-time streaming
- Multiple language support
- Custom vocabulary support
- Speaker diarization

**Configuration:**
```bash
# Environment Variables
ASR_URL=http://avr-asr-deepgram:6001/speech-to-text-stream
DEEPGRAM_API_KEY=your_api_key
```

**Docker Compose:**
```yaml
services:
  avr-asr-deepgram:
    image: agentvoiceresponse/avr-asr-deepgram
    environment:
      - PORT=6001
      - DEEPGRAM_API_KEY=${DEEPGRAM_API_KEY}
      - LANGUAGE=en-US
      - MODEL=nova-2
```

**Example Usage:**
```javascript
// Streaming audio to Deepgram
const stream = await deepgram.transcription.stream({
  punctuate: true,
  model: 'nova-2',
  language: 'en-US',
  encoding: 'linear16',
  sample_rate: 16000
});

stream.on('data', (data) => {
  console.log(data.channel.alternatives[0].transcript);
});
```

### 2. Google Cloud Speech (`avr-asr-google-cloud-speech`)

**Features:**
- Google's speech recognition
- High accuracy
- Multiple language support
- Cloud-based service
- Custom models support

**Configuration:**
```bash
# Environment Variables
ASR_URL=http://avr-asr-google-cloud-speech:6001/speech-to-text-stream
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

**Docker Compose:**
```yaml
services:
  avr-asr-google:
    image: agentvoiceresponse/avr-asr-google-cloud-speech
    environment:
      - PORT=6001
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json
      - LANGUAGE_CODE=en-US
      - MODEL=default
    volumes:
      - ./credentials.json:/app/credentials.json
```

**Example Usage:**
```javascript
// Streaming audio to Google Cloud Speech
const stream = speechClient.streamingRecognize({
  config: {
    encoding: 'LINEAR16',
    sampleRateHertz: 16000,
    languageCode: 'en-US',
    model: 'default'
  }
});

stream.on('data', (data) => {
  console.log(data.results[0].alternatives[0].transcript);
});
```

## Integration Guide

### 1. Choose a Provider

Select an ASR provider based on your requirements:
- Deepgram for high accuracy and custom vocabulary
- Google Cloud Speech for Google ecosystem integration

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

## Related Documentation

- [Speech To Text Integrations](./integrations/speech-to-text.md) - For providers that don't support direct ASR integration