# Quick Start Guide

## Overview

This guide provides step-by-step instructions for setting up common AVR configurations quickly.

## 1. Basic Setup with Silero VAD

### Prerequisites
- Docker and Docker Compose installed
- Basic understanding of environment variables

### Steps

1. **Clone the Infrastructure**
   ```bash
   git clone https://github.com/agentvoiceresponse/avr-infra.git
   cd avr-infra
   ```

2. **Create Environment File**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. **Start Core Services**
   ```bash
   docker-compose -f docker-compose.yml up -d
   ```

4. **Start Silero VAD**
   ```bash
   docker-compose -f docker-compose-silero.yml up -d
   ```

## 2. Customer Service Bot

### Prerequisites
- OpenAI API key
- ElevenLabs API key

### Steps

1. **Configure Environment**
   ```bash
   # .env
   OPENAI_API_KEY=your_openai_key
   ELEVENLABS_API_KEY=your_elevenlabs_key
   ```

2. **Start Services**
   ```bash
   # Start core
   docker-compose -f docker-compose.yml up -d
   
   # Start OpenAI LLM
   docker-compose -f docker-compose-openai.yml up -d
   
   # Start ElevenLabs TTS
   docker-compose -f docker-compose-elevenlabs.yml up -d
   ```

3. **Test the Setup**
   ```bash
   curl -X POST http://localhost:3000/test \
     -H "Content-Type: application/json" \
     -d '{"message": "Hello, how can I help you?"}'
   ```

## 3. Multi-Language Support

### Prerequisites
- Google Cloud credentials
- OpenRouter API key

### Steps

1. **Configure Environment**
   ```bash
   # .env
   GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
   OPENROUTER_API_KEY=your_openrouter_key
   ```

2. **Start Services**
   ```bash
   # Start core
   docker-compose -f docker-compose.yml up -d
   
   # Start Google ASR
   docker-compose -f docker-compose-gemini.yml up -d
   
   # Start OpenRouter LLM
   docker-compose -f docker-compose-openrouter.yml up -d
   
   # Start Google TTS
   docker-compose -f docker-compose-google-cloud-tts.yml up -d
   ```

3. **Configure Languages**
   ```yaml
   # docker-compose.yml
   services:
     avr-core:
       environment:
         - SUPPORTED_LANGUAGES=en,es,fr,de
   ```

## 4. Visual Conversation Builder

### Prerequisites
- Typebot account
- Deepgram API key

### Steps

1. **Configure Environment**
   ```bash
   # .env
   TYPEBOT_API_KEY=your_typebot_key
   TYPEBOT_ID=your_typebot_id
   DEEPGRAM_API_KEY=your_deepgram_key
   ```

2. **Start Services**
   ```bash
   # Start core
   docker-compose -f docker-compose.yml up -d
   
   # Start Typebot
   docker-compose -f docker-compose-typebot.yml up -d
   
   # Start Deepgram TTS
   docker-compose -f docker-compose-deepgram.yml up -d
   ```

3. **Design Conversation Flow**
   - Access Typebot dashboard
   - Create conversation flows
   - Test the integration

## Common Issues and Solutions

### 1. Service Connection Issues
```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs avr-core
```

### 2. Audio Quality Problems
```yaml
# Adjust audio settings
services:
  avr-core:
    environment:
      - AUDIO_SAMPLE_RATE=16000
      - AUDIO_CHANNELS=1
      - AUDIO_BITRATE=16
```

### 3. Performance Issues
```yaml
# Optimize resource allocation
services:
  avr-core:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

## Next Steps

1. **Monitor Performance**
   - Set up monitoring tools
   - Configure alerts
   - Review logs

2. **Scale Services**
   - Add more replicas
   - Configure load balancing
   - Implement caching

3. **Enhance Security**
   - Configure SSL/TLS
   - Set up authentication
   - Implement rate limiting

## Support

<<<<<<< HEAD
For quick start issues:
- Check the [documentation](https://wiki.agentvoiceresponse.com/)
- Review service logs
- Contact support at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com)

## Related Documentation

- [Specific Use Cases](Specific Use Cases.md) - For more detailed use cases
- [Provider Combinations](Provider Combinations Comparison.md) - For provider options
- [Performance Optimization](../Advanced/Performance Optimization Guide.md) - For performance tuning
- [Security Best Practices](../Advanced/Security Best Practices.md) - For security setup 
=======
For additional help:
- Check the [documentation](https://github.com/agentvoiceresponse/docs)
- Join the [community](https://github.com/agentvoiceresponse/discussions)
- Contact [support](mailto:info@agentvoiceresponse.com) 
>>>>>>> d38d9d88b718ecddf7d4a27f095a0764d7fe1b1d
