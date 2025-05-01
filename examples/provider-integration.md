# Provider Integration Examples

This guide provides examples of how to integrate different providers with AVR, including ASR, LLM, and TTS services.

## ASR Integration Examples

### 1. ElevenLabs STT Integration

```javascript
const express = require('express');
const app = express();
const { ElevenLabsClient } = require('elevenlabs-node');

const client = new ElevenLabsClient({
  apiKey: process.env.ELEVENLABS_API_KEY
});

app.post('/speech-to-text-stream', async (req, res) => {
  try {
    const audioStream = req.body;
    const transcription = await client.transcribe(audioStream);
    res.json({ text: transcription });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6022);
```

### 2. Google Cloud STT Integration

```javascript
const express = require('express');
const app = express();
const speech = require('@google-cloud/speech');

const client = new speech.SpeechClient();

app.post('/speech-to-text-stream', async (req, res) => {
  try {
    const audioStream = req.body;
    const [response] = await client.recognize({
      audio: {
        content: audioStream,
      },
      config: {
        encoding: 'LINEAR16',
        sampleRateHertz: 16000,
        languageCode: 'en-US',
      },
    });
    res.json({ text: response.results[0].alternatives[0].transcript });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6021);
```

## LLM Integration Examples

### 1. OpenAI Integration

```javascript
const express = require('express');
const app = express();
const OpenAI = require('openai');

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

app.post('/prompt-stream', async (req, res) => {
  try {
    const { messages } = req.body;
    const stream = await openai.chat.completions.create({
      model: 'gpt-3.5-turbo',
      messages,
      stream: true,
    });

    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    for await (const chunk of stream) {
      res.write(`data: ${JSON.stringify(chunk)}\n\n`);
    }
    res.end();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6002);
```

### 2. Anthropic Integration

```javascript
const express = require('express');
const app = express();
const Anthropic = require('@anthropic-ai/sdk');

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

app.post('/prompt-stream', async (req, res) => {
  try {
    const { messages } = req.body;
    const stream = await anthropic.messages.create({
      model: 'claude-3-haiku-20240307',
      messages,
      stream: true,
    });

    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    for await (const chunk of stream) {
      res.write(`data: ${JSON.stringify(chunk)}\n\n`);
    }
    res.end();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6014);
```

## TTS Integration Examples

### 1. Google Cloud TTS Integration

```javascript
const express = require('express');
const app = express();
const textToSpeech = require('@google-cloud/text-to-speech');

const client = new textToSpeech.TextToSpeechClient();

app.post('/text-to-speech-stream', async (req, res) => {
  try {
    const { text } = req.body;
    const [response] = await client.synthesizeSpeech({
      input: { text },
      voice: {
        languageCode: 'en-US',
        name: 'en-US-Neural2-C',
      },
      audioConfig: {
        audioEncoding: 'LINEAR16',
        sampleRateHertz: 8000,
      },
    });

    res.setHeader('Content-Type', 'audio/wav');
    res.send(response.audioContent);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6003);
```

### 2. ElevenLabs TTS Integration

```javascript
const express = require('express');
const app = express();
const { ElevenLabsClient } = require('elevenlabs-node');

const client = new ElevenLabsClient({
  apiKey: process.env.ELEVENLABS_API_KEY
});

app.post('/text-to-speech-stream', async (req, res) => {
  try {
    const { text } = req.body;
    const audioStream = await client.textToSpeech({
      text,
      voice_id: process.env.ELEVENLABS_VOICE_ID,
      model_id: 'eleven_monolingual_v1',
    });

    res.setHeader('Content-Type', 'audio/wav');
    audioStream.pipe(res);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(process.env.PORT || 6007);
```

## Docker Configuration Examples

### 1. Basic Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 6000

CMD ["node", "src/index.js"]
```

### 2. Docker Compose Configuration

```yaml
version: '3.8'

services:
  avr-provider:
    image: your-provider-image
    container_name: avr-provider
    environment:
      - PORT=6000
      - API_KEY=${PROVIDER_API_KEY}
    ports:
      - "6000:6000"
    restart: unless-stopped
```

## Environment Variables

### 1. ASR Provider

```bash
# Server Settings
PORT=6000

# Provider Settings
PROVIDER_API_KEY=your_api_key
PROVIDER_MODEL=default_model
PROVIDER_LANGUAGE=en
```

### 2. LLM Provider

```bash
# Server Settings
PORT=6000

# Provider Settings
PROVIDER_API_KEY=your_api_key
PROVIDER_MODEL=gpt-3.5-turbo
PROVIDER_TEMPERATURE=0.7
PROVIDER_MAX_TOKENS=100
```

### 3. TTS Provider

```bash
# Server Settings
PORT=6000

# Provider Settings
PROVIDER_API_KEY=your_api_key
PROVIDER_VOICE_ID=default_voice
PROVIDER_MODEL=default_model
PROVIDER_LANGUAGE=en
```

## Error Handling Examples

### 1. Basic Error Handling

```javascript
try {
  // Provider integration logic
} catch (error) {
  console.error('Provider error:', error);
  res.status(500).json({
    error: 'Internal server error',
    message: error.message
  });
}
```

### 2. Advanced Error Handling

```javascript
const handleError = (error, res) => {
  console.error('Provider error:', error);
  
  if (error.response) {
    // Provider API error
    res.status(error.response.status).json({
      error: 'Provider API error',
      message: error.response.data.message
    });
  } else if (error.request) {
    // Network error
    res.status(503).json({
      error: 'Service unavailable',
      message: 'Could not reach provider API'
    });
  } else {
    // Other errors
    res.status(500).json({
      error: 'Internal server error',
      message: error.message
    });
  }
};

app.post('/endpoint', async (req, res) => {
  try {
    // Provider integration logic
  } catch (error) {
    handleError(error, res);
  }
});
```

## Testing Examples

### 1. Unit Tests

```javascript
const test = require('ava');
const { transcribe } = require('./src/transcribe');

test('transcribe audio', async t => {
  const audio = Buffer.from('test audio data');
  const result = await transcribe(audio);
  t.is(typeof result, 'string');
});

test('handle invalid audio', async t => {
  const audio = null;
  await t.throwsAsync(() => transcribe(audio));
});
```

### 2. Integration Tests

```javascript
const test = require('ava');
const request = require('supertest');
const app = require('./src/app');

test('speech-to-text endpoint', async t => {
  const response = await request(app)
    .post('/speech-to-text-stream')
    .send({ audio: 'test audio data' });
  
  t.is(response.status, 200);
  t.is(typeof response.body.text, 'string');
});
```

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