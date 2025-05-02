---
title: development-guide
description: 
published: true
date: 2025-05-02T16:10:45.598Z
tags: 
editor: markdown
dateCreated: 2025-05-01T19:09:48.761Z
---

# AVR Integration Development Guide

This guide will help you understand how to develop new integrations for AVR (Agent Voice Response). Whether you want to add support for a new provider or create a custom integration, this guide will walk you through the process.

## Integration Types

AVR supports three main types of integrations:

1. **Automatic Speech Recognition (ASR)**
   - Converts speech to text
   - Handles real-time audio streaming
   - Supports multiple languages
   - Example: [Deepgram ASR](examples/deepgram-stt.md)
   
2. **Speech To Text (STT)**
   - Converts speech to text
   - Handles real-time audio streaming
   - Supports multiple languages
   - Example: [ElevenLabs STT](examples/elevenlabs-stt.md)
   
3. **Large Language Model (LLM)**
   - Processes text input
   - Generates intelligent responses
   - Manages conversation context
   - Example: [OpenAI LLM](examples/openai-llm.md)

4. **Text-to-Speech (TTS)**
   - Converts text to speech
   - Supports multiple voices
   - Handles different languages
   - Example: [Google Cloud TTS](examples/google-cloud-tts.md)


## Development Requirements

To develop an AVR integration, you'll need:

1. **Node.js and npm**
   - Latest LTS version recommended
   - Basic understanding of Express.js

2. **Docker**
   - For containerization
   - For testing and deployment

3. **API Access**
   - Access to the provider's API
   - API keys and credentials

## Integration Structure

Each integration should follow this basic structure:

```
avr-provider-[name]/
├── src/
│   ├── index.js
│   ├── config.js
│   └── utils.js
├── Dockerfile
├── docker-compose.yml
├── package.json
├── .env.example
└── README.md
```

### Required Files

1. **index.js**
   ```javascript
   const express = require('express');
   const app = express();
   
   // Required endpoints
   app.post('/speech-to-text-stream', handleSpeechToText);
   app.post('/prompt-stream', handlePrompt);
   app.post('/text-to-speech-stream', handleTextToSpeech);
   
   // Start server
   app.listen(process.env.PORT || 6000);
   ```

2. **Dockerfile**
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /usr/src/app
   COPY package*.json ./
   RUN npm install
   COPY . .
   CMD ["node", "src/index.js"]
   ```

3. **docker-compose.yml**
   ```yaml
   version: '3.8'
   services:
     avr-provider:
       image: your-provider-image
       environment:
         - PORT=6000
         - API_KEY=${PROVIDER_API_KEY}
       ports:
         - "6000:6000"
   ```

## Integration Guidelines

### 1. Environment Variables

Define all required environment variables in `.env.example`:

```bash
# Server Settings
PORT=6000

# Provider Settings
PROVIDER_API_KEY=your_api_key
PROVIDER_MODEL=default_model
PROVIDER_LANGUAGE=en
```

### 2. API Endpoints

Each integration must implement these standard endpoints:

#### ASR Integration
```javascript
app.post('/speech-to-text-stream', async (req, res) => {
  // Handle audio stream
  // Convert to text
  // Return transcription
});
```

#### LLM Integration
```javascript
app.post('/prompt-stream', async (req, res) => {
  // Handle text input
  // Process with LLM
  // Stream response
});
```

#### TTS Integration
```javascript
app.post('/text-to-speech-stream', async (req, res) => {
  // Handle text input
  // Convert to speech
  // Stream audio
});
```

### 3. Error Handling

Implement proper error handling:

```javascript
try {
  // Integration logic
} catch (error) {
  console.error('Integration error:', error);
  res.status(500).json({
    error: 'Internal server error',
    message: error.message
  });
}
```

### 4. Testing

Create test cases for your integration:

```javascript
const test = require('ava');

test('speech-to-text conversion', async t => {
  // Test implementation
});

test('text-to-speech conversion', async t => {
  // Test implementation
});
```

## Asterisk Integration

### Basic Setup

To integrate with Asterisk, you'll need:

1. **Asterisk Configuration**
   ```ini
   [from-internal]
   exten => _6XXX,1,Answer()
   exten => _6XXX,n,AudioSocket(127.0.0.1,6000)
   exten => _6XXX,n,Hangup()
   ```

2. **AudioSocket Application**
   - Always use `Answer()` before `AudioSocket()`
   - Configure correct IP and port
   - Handle audio format conversion

### FreePBX Integration

To integrate with FreePBX:

1. **Install AudioSocket Module**
   ```bash
   cd /usr/src
   wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
   tar xvf asterisk-20-current.tar.gz
   cd asterisk-20.*/contrib/ast_tools
   make
   ```

2. **Configure FreePBX**
   ```ini
   [from-internal-custom]
   exten => _6XXX,1,Answer()
   exten => _6XXX,n,AudioSocket(127.0.0.1,6000)
   exten => _6XXX,n,Hangup()
   ```

3. **Create Custom Destination**
   - Go to FreePBX Admin
   - Create new Custom Destination
   - Point to your AVR integration

## Best Practices

1. **Error Handling**
   - Implement comprehensive error handling
   - Log errors appropriately
   - Provide meaningful error messages

2. **Performance**
   - Optimize audio processing
   - Implement caching where appropriate
   - Monitor resource usage

3. **Security**
   - Secure API keys
   - Implement rate limiting
   - Validate input data

4. **Documentation**
   - Document all endpoints
   - Provide usage examples
   - Include configuration guides

## Example Integrations

### 1. Basic ASR Integration
```javascript
const express = require('express');
const app = express();

app.post('/speech-to-text-stream', async (req, res) => {
  try {
    const audioStream = req.body;
    const transcription = await providerAPI.transcribe(audioStream);
    res.json({ text: transcription });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6000);
```

### 2. Basic TTS Integration
```javascript
const express = require('express');
const app = express();

app.post('/text-to-speech-stream', async (req, res) => {
  try {
    const { text } = req.body;
    const audioStream = await providerAPI.synthesize(text);
    res.setHeader('Content-Type', 'audio/wav');
    audioStream.pipe(res);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6000);
```

## Next Steps

1. **Testing**
   - Test your integration thoroughly
   - Verify Asterisk compatibility
   - Check performance under load

2. **Documentation**
   - Create detailed documentation
   - Include setup instructions
   - Provide usage examples

3. **Deployment**
   - Containerize your integration
   - Set up monitoring
   - Configure logging

4. **Maintenance**
   - Regular updates
   - Bug fixes
   - Performance optimization 