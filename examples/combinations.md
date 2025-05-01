<<<<<<< HEAD
=======
---
title: combinations
description: 
published: true
date: 2025-05-01T18:05:19.469Z
tags: 
editor: markdown
dateCreated: 2025-05-01T18:05:14.906Z
---

>>>>>>> d38d9d88b718ecddf7d4a27f095a0764d7fe1b1d
# Example Provider Combinations

This document provides practical examples of combining different ASR, LLM, and TTS providers to create optimal voice response systems.

## 1. High-Quality English Support

### Components
- **ASR**: ElevenLabs (high accuracy for English)
- **LLM**: Anthropic Claude (excellent response quality)
- **TTS**: ElevenLabs (realistic voices)

### Configuration
```yaml
# docker-compose.yml
services:
  avr-asr:
    image: agentvoiceresponse/avr-stt-elevenlabs
    environment:
      - PORT=6001
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}

  avr-llm:
    image: agentvoiceresponse/avr-llm-anthropic
    environment:
      - PORT=6005
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - MODEL_NAME=claude-3-opus-20240229

  avr-tts:
    image: agentvoiceresponse/avr-tts-elevenlabs
    environment:
      - PORT=6003
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

### Use Case
Ideal for:
- Customer service applications
- High-end virtual assistants
- Professional voice interfaces

## 2. Cost-Effective Multi-Language Support

### Components
- **ASR**: Google Cloud Speech-to-Text (wide language support)
- **LLM**: OpenRouter (cost-effective with multiple models)
- **TTS**: Google Cloud TTS (extensive language support)

### Configuration
```yaml
# docker-compose.yml
services:
  avr-asr:
    image: agentvoiceresponse/avr-stt-gemini
    environment:
      - PORT=6001
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json

  avr-llm:
    image: agentvoiceresponse/avr-llm-openrouter
    environment:
      - PORT=6005
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - MODEL_NAME=anthropic/claude-3-opus

  avr-tts:
    image: agentvoiceresponse/avr-tts-google-cloud-tts
    environment:
      - PORT=6003
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json
      - VOICE_NAME=${VOICE_NAME}
```

### Use Case
Ideal for:
- International customer support
- Multi-language applications
- Budget-conscious deployments

## 3. Japanese-Specific Solution

### Components
- **ASR**: Google Cloud Speech-to-Text (Japanese support)
- **LLM**: OpenAI (with Japanese language model)
- **TTS**: Kokoro (Japanese-specific voices)

### Configuration
```yaml
# docker-compose.yml
services:
  avr-asr:
    image: agentvoiceresponse/avr-stt-gemini
    environment:
      - PORT=6001
      - GOOGLE_APPLICATION_CREDENTIALS=/app/credentials.json
      - LANGUAGE_CODE=ja-JP

  avr-llm:
    image: agentvoiceresponse/avr-llm-openai
    environment:
      - PORT=6005
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - MODEL_NAME=gpt-4-turbo-preview

  avr-tts:
    image: agentvoiceresponse/avr-tts-kokoro
    environment:
      - PORT=6003
      - KOKORO_API_KEY=${KOKORO_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

### Use Case
Ideal for:
- Japanese customer service
- Japanese language learning
- Japan-specific applications

## 4. Conversational Flow with Visual Builder

### Components
- **ASR**: Silero VAD (open-source)
- **LLM**: Typebot (visual conversation builder)
- **TTS**: Deepgram (high-quality voices)

### Configuration
```yaml
# docker-compose.yml
services:
  avr-asr:
    image: agentvoiceresponse/avr-asr-to-stt
    environment:
      - PORT=6001
      - MODEL_PATH=/app/silero_vad.onnx

  avr-llm:
    image: agentvoiceresponse/avr-llm-typebot
    environment:
      - PORT=6005
      - TYPEBOT_API_KEY=${TYPEBOT_API_KEY}
      - TYPEBOT_ID=${TYPEBOT_ID}

  avr-tts:
    image: agentvoiceresponse/avr-tts-deepgram
    environment:
      - PORT=6003
      - DEEPGRAM_API_KEY=${DEEPGRAM_API_KEY}
      - VOICE_ID=${VOICE_ID}
```

### Use Case
Ideal for:
- Visual conversation design
- Rapid prototyping
- Open-source solutions

## Best Practices for Combining Providers

1. **Consistency**
   - Ensure all providers support the required languages
   - Match voice quality across components
   - Maintain consistent response times

2. **Cost Optimization**
   - Balance quality and cost across components
   - Use appropriate quality settings
   - Implement caching strategies

3. **Performance**
   - Monitor latency across the entire pipeline
   - Implement proper error handling
   - Use appropriate buffer sizes

4. **Maintenance**
   - Keep API keys secure
   - Monitor service health
   - Implement proper logging

## Troubleshooting Combined Setups

1. **Latency Issues**
   - Check each component's response time
   - Verify network connectivity
   - Review buffer settings

2. **Quality Problems**
   - Test each component independently
   - Verify language support
   - Check audio quality settings

3. **Integration Errors**
   - Review API documentation
   - Check service health
   - Verify configuration settings 