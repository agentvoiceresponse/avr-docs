---
title: development-guide
description: 
published: true
date: 2025-05-02T16:21:13.362Z
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
├── index.js
├── Dockerfile
├── docker-compose.yml
├── package.json
├── .env.example
├── README.md
└── LICENSE.md
```

### Required Files

1. **index.js**
   ```javascript
   const express = require('express');
   const app = express();
   
   // Required endpoints

   // ASR
   app.post('/speech-to-text-stream', handleSpeechToText);

   // STT (Remember that you need avr-asr-to-stt)
   app.post('/transcribe', handleTranscriptionRequest);
   
   // LLM
   app.post('/prompt-stream', handlePrompt);

   // TTS
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
   CMD ["node", "index.js"]
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
  res.write('Hi, my name is John Doe');
});
```

#### STT Integration
```javascript
app.post('/transcribe', async (req, res) => {
  // Handle audio stream
  // Convert to text
  // Return transcription
  return res.json({ transcription: 'Hello John Doe, How can I help you?' });
});
```

#### LLM Integration
```javascript
app.post('/prompt-stream', async (req, res) => {
  // Handle text input
  // Process with LLM
  // Stream response
  res.write(JSON.stringify({ 
    type: 'text', 
    content: 'Hello John Doe, How can I help you?' 
  }));
});
```

#### TTS Integration
```javascript
app.post('/text-to-speech-stream', async (req, res) => {
  // Handle text input
  // Convert to speech
  // Stream audio
  res.write(audioChunk);
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
   exten => _6XXX,n,Ringing()
	 exten => _6XXX,n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
	 exten => _6XXX,n,AudioSocket(${UUID},AVR_HOST_IP_OR_DOMAIN:AVR_HOST_PORT)
   exten => _6XXX,n,Hangup()
   ```

2. **AudioSocket Application**
   - Always use `Answer()` before `AudioSocket()`
   - Configure correct IP and port
   - Handle audio format conversion

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

### 1. Deepgram ASR Integration
```javascript
/**
 * index.js
 * This file is the main entry point for the application using Deepgram Speech-to-Text.
 * @author  Giuseppe Careri
 * @see https://www.gcareri.com
 */
const express = require('express');
const { Writable } = require('stream');
const { createClient, LiveTranscriptionEvents } = require('@deepgram/sdk');
require('dotenv').config();

const app = express();
const deepgram = createClient(process.env.DEEPGRAM_API_KEY);

console.log("Deepgram Speech Configuration Loaded");

/**
 * Handles an audio stream from the client and uses Deepgram API
 * to recognize the speech and stream the transcript back to the client.
 *
 * @param {Object} req - The Express request object
 * @param {Object} res - The Express response object
 */
const handleAudioStream = async (req, res) => {
  try {
    const connection = await deepgram.listen.live({
      model: process.env.SPEECH_RECOGNITION_MODEL || 'nova',
      language: process.env.SPEECH_RECOGNITION_LANGUAGE || 'en',
      interim_results: true,
      // smart_format: true,
      // channels: 2,
      encoding: 'linear16',
      sample_rate: 8000
    });

    connection.addListener(LiveTranscriptionEvents.Open, () => {
      console.log('Deepgram Connection Opened')

      connection.addListener(LiveTranscriptionEvents.Close, () => {
        console.log('Deepgram Connection Closed');
        res.end();
      });

      connection.addListener(LiveTranscriptionEvents.Transcript, (data) => {
        const transcript = data.channel.alternatives[0].transcript;
        if (transcript) {
          console.log(`${transcript}...`);
        }

        if (data.speech_final) {
          // Remove "e" for italian bug
          if (transcript && transcript !== 'e') {
            console.log(`Transcript: ${transcript}`);
            res.write(transcript);
          } else {
            console.log('No transcript available');
          }
        }
      });

      connection.on(LiveTranscriptionEvents.Metadata, (data) => {
        console.log(data);
      });

      connection.on(LiveTranscriptionEvents.Warning, async (warning) => {
        console.log(warning);
      });

      connection.on(LiveTranscriptionEvents.Error, (error) => {
        console.error('Deepgram API Error:', error);
        res.status(500).json({ message: 'Deepgram API error' });
      });
    });

    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    req.on('data', (chunk) => {
      connection.send(chunk)
    });

    req.on('end', () => connection.requestClose());
    req.on('error', (err) => {
      console.error('Error receiving audio stream:', err);
      req.destroy();
      res.status(500).json({ message: 'Error receiving audio stream' });
    });
  } catch (err) {
    console.error('Error handling audio stream:', err);
    res.status(500).json({ message: err.message });
  }
};

app.post('/speech-to-text-stream', handleAudioStream);

const port = process.env.PORT || 6010;
app.listen(port, () => {
  console.log(`Deepgram Audio endpoint listening on port ${port}`);
});
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