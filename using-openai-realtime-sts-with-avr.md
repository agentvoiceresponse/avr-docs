---
title: OpenAI Realtime Speech-to-Speech
description: 
published: true
date: 2025-09-04T15:05:27.174Z
tags: 
editor: markdown
dateCreated: 2025-09-04T15:05:27.174Z
---

# OpenAI Realtime Speech-to-Speech (STS)

The **OpenAI Realtime STS integration** enables AVR to leverage OpenAI’s real-time speech-to-speech API.  
This mode provides low-latency voice conversations, with the added benefit of **function calling support** for advanced integrations.  

👉 For more details on function calls, see [AVR Function Calls](https://wiki.agentvoiceresponse.com/en/avr-function-calls).

---

## Environment Variables

| Variable              | Description                                           | Example Value                           |
|-----------------------|-------------------------------------------------------|-----------------------------------------|
| PORT                  | Port on which the STS service runs                    | 6030                                    |
| OPENAI_API_KEY        | Your OpenAI API key                                   | sk-xxxxxx                               |
| OPENAI_MODEL          | OpenAI model ID to use                                | gpt-4o-realtime-preview                 |
| OPENAI_INSTRUCTIONS   | System prompt for the assistant                       | "You are a helpful assistant."          |
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
    - STS_URL=http://avr-sts-openai:6030/speech-to-speech-stream
  ports:
    - 5001:5001
  networks:
    - avr
```

For a complete example, check the docker-compose-openai.yml in the [avr-infra](https://github.com/agentvoiceresponse/avr-infra) repository.