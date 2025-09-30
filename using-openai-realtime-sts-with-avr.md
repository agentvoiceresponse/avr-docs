---
title: OpenAI Realtime Speech-to-Speech
description: 
published: true
date: 2025-09-30T11:30:20.397Z
tags: sts, openai
editor: markdown
dateCreated: 2025-09-04T15:05:27.174Z
---

# OpenAI Realtime Speech-to-Speech (STS)

The **OpenAI Realtime STS integration** enables AVR to leverage OpenAI’s real-time speech-to-speech API.  
This mode provides low-latency voice conversations, with the added benefit of **function calling support** for advanced integrations.  

For more details on function calls, see [AVR Function Calls](https://wiki.agentvoiceresponse.com/en/avr-function-calls).

---

## Environment Variables

| Variable              | Description                                           | Example Value                           |
|-----------------------|-------------------------------------------------------|-----------------------------------------|
| PORT                  | Port on which the STS service runs                    | 6030                                    |
| OPENAI_API_KEY        | Your OpenAI API key                                   | sk-xxxxxx                               |
| OPENAI_MODEL          | OpenAI model ID to use                                | gpt-4o-realtime-preview                 |
| OPENAI_INSTRUCTIONS   | # Method 1: Direct variable                       | "You are a helpful assistant."          |
| *OPENAI_URL_INSTRUCTIONS   | # Method 2: Web service                       | https://your-api.com/instructions          |
| *OPENAI_FILE_INSTRUCTIONS   | # Method 3: Local file                       | ./instructions.txt          |
| OPENAI_TEMPERATURE    | Controls randomness in responses (0.0–1.0, default 0.8) | 0.8                                     |
| OPENAI_MAX_TOKENS     | Maximum response length (default: unlimited)          | 100                                     |

---

## Docker Compose Example

```yaml
avr-sts-openai:
  image: agentvoiceresponse/avr-sts-openai
  platform: linux/x86_64
  container_name: avr-sts-openai
  restart: always
  environment:
    - PORT=6030
    - OPENAI_API_KEY=$OPENAI_API_KEY
    - OPENAI_MODEL=gpt-4o-realtime-preview
    - OPENAI_INSTRUCTIONS="You are a helpful assistant."
    - OPENAI_TEMPERATURE=0.8
    - OPENAI_MAX_TOKENS=100
  networks:
    - avr
```

## Integration with AVR Core

Update your avr-core service to point to the OpenAI STS endpoint:

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    - STS_URL=ws://avr-sts-openai:6030
  ports:
    - 5001:5001
  networks:
    - avr
```

### Instruction Loading Methods

The application supports three different methods for loading AI instructions, with a specific priority order:

#### 1. Environment Variable (Highest Priority)
Set the `OPENAI_INSTRUCTIONS` environment variable with your custom instructions:

```bash
OPENAI_INSTRUCTIONS="You are a specialized customer service agent for a tech company. Always be polite and helpful."
```

#### 2. Web Service (Medium Priority)
If no environment variable is set, the application can fetch instructions from a web service using the `OPENAI_URL_INSTRUCTIONS` environment variable:

```bash
OPENAI_URL_INSTRUCTIONS=https://your-api.com/instructions
```

The web service should return a JSON response with a `system` field containing the instructions:
```json
{
  "system": "You are a helpful assistant that provides technical support."
}
```

The application will include the session UUID in the request headers as `X-AVR-UUID` for personalized instructions.

#### 3. File (Lowest Priority)
If neither environment variable nor web service is configured, the application can load instructions from a local file using the `OPENAI_FILE_INSTRUCTIONS` environment variable:

```bash
OPENAI_FILE_INSTRUCTIONS=./instructions.txt
```

The file should contain plain text instructions that will be used as the system prompt.

For a complete example, check the docker-compose-openai-realtime.yml in the [avr-infra](https://github.com/agentvoiceresponse/avr-infra) repository.