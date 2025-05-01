---
title: llm
description: 
published: true
date: 2025-05-01T18:05:52.026Z
tags: 
editor: markdown
dateCreated: 2025-05-01T18:05:46.484Z
---

# LLM (Large Language Model) Integrations

## Overview

AVR supports multiple LLM providers for natural language understanding and response generation. Each provider offers different capabilities in terms of model size, response quality, and features.

## Available Providers

### 1. Anthropic (`avr-llm-anthropic`)

**Features:**
- Claude models
- High-quality responses
- Contextual understanding
- Commercial API

**Configuration:**
```bash
# Environment Variables
LLM_URL=http://localhost:6005/prompt-stream
ANTHROPIC_API_KEY=your_api_key
MODEL_NAME=claude-3-opus-20240229
```

**Docker Compose:**
```yaml
services:
  avr-llm-anthropic:
    image: agentvoiceresponse/avr-llm-anthropic
    environment:
      - PORT=6005
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - MODEL_NAME=${MODEL_NAME}
```

### 2. OpenAI (`avr-llm-openai`)

**Features:**
- GPT models
- Wide range of capabilities
- Extensive documentation
- Commercial API

**Configuration:**
```bash
# Environment Variables
LLM_URL=http://localhost:6005/prompt-stream
OPENAI_API_KEY=your_api_key
MODEL_NAME=gpt-4-turbo-preview
```

**Docker Compose:**
```yaml
services:
  avr-llm-openai:
    image: agentvoiceresponse/avr-llm-openai
    environment:
      - PORT=6005
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - MODEL_NAME=${MODEL_NAME}
```

### 3. OpenAI Assistant (`avr-llm-openai-assistant`)

**Features:**
- Assistant API integration
- Persistent context
- File handling
- Tool usage

**Configuration:**
```bash
# Environment Variables
LLM_URL=http://localhost:6005/prompt-stream
OPENAI_API_KEY=your_api_key
ASSISTANT_ID=your_assistant_id
```

**Docker Compose:**
```yaml
services:
  avr-llm-openai-assistant:
    image: agentvoiceresponse/avr-llm-openai-assistant
    environment:
      - PORT=6005
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ASSISTANT_ID=${ASSISTANT_ID}
```

### 4. OpenRouter (`avr-llm-openrouter`)

**Features:**
- Multiple model providers
- Cost-effective
- Unified API
- Commercial service

**Configuration:**
```bash
# Environment Variables
LLM_URL=http://localhost:6005/prompt-stream
OPENROUTER_API_KEY=your_api_key
MODEL_NAME=anthropic/claude-3-opus
```

**Docker Compose:**
```yaml
services:
  avr-llm-openrouter:
    image: agentvoiceresponse/avr-llm-openrouter
    environment:
      - PORT=6005
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - MODEL_NAME=${MODEL_NAME}
```

### 5. Typebot (`avr-llm-typebot`)

**Features:**
- Conversational flows
- Visual builder
- Custom logic
- Open-source

**Configuration:**
```bash
# Environment Variables
LLM_URL=http://localhost:6005/prompt-stream
TYPEBOT_API_KEY=your_api_key
TYPEBOT_ID=your_typebot_id
```

**Docker Compose:**
```yaml
services:
  avr-llm-typebot:
    image: agentvoiceresponse/avr-llm-typebot
    environment:
      - PORT=6005
      - TYPEBOT_API_KEY=${TYPEBOT_API_KEY}
      - TYPEBOT_ID=${TYPEBOT_ID}
```

## Integration Guide

### 1. Choose a Provider

Select an LLM provider based on your requirements:
- For high-quality responses: Anthropic or OpenAI
- For cost-effective solutions: OpenRouter
- For conversational flows: Typebot
- For persistent context: OpenAI Assistant

### 2. Configure Environment

1. Set up the required environment variables
2. Configure API keys
3. Set up any required model parameters

### 3. Start the Service

```bash
# Using Docker Compose
docker-compose -f docker-compose-{provider}.yml up -d
```

### 4. Test the Integration

```bash
# Example curl command to test the service
curl -X POST http://localhost:6005/prompt-stream \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello, how can I help you?"}'
```

## Best Practices

1. **Prompt Engineering**
   - Use clear, concise prompts
   - Include context when necessary
   - Set appropriate temperature and other parameters

2. **Performance**
   - Monitor response times
   - Implement caching for common queries
   - Handle rate limits appropriately

3. **Cost Management**
   - Monitor token usage
   - Implement conversation history limits
   - Use appropriate model sizes

## Troubleshooting

### Common Issues

1. **Slow Responses**
   - Check network connectivity
   - Verify API rate limits
   - Monitor model load

2. **Poor Quality Responses**
   - Review prompt engineering
   - Check model parameters
   - Verify context handling

3. **API Errors**
   - Verify API keys
   - Check service health
   - Review error logs

## Support

For issues specific to LLM integrations:
- Check provider documentation
- Review service logs
- Contact support at [info@agentvoiceresponse.com](mailto:info@agentvoiceresponse.com) 