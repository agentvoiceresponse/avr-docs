---
title: ElevenLabs Speech To Speech Integration
description: 
published: true
date: 2025-12-15T08:58:53.959Z
tags: 
editor: markdown
dateCreated: 2025-09-05T16:53:25.321Z
---

# ElevenLabs Speech-to-Speech Integration

The Agent Voice Response (AVR) platform supports integration with ElevenLabs Speech-to-Speech (STS), enabling high-quality, real-time conversational AI. ElevenLabs is well known for its natural and expressive voices, making it an excellent choice for creating human-like conversational agents.

## Why Use ElevenLabs STS?
- Natural, Human-like Voices → ElevenLabs offers some of the most realistic voice synthesis in the industry.
- Real-time Streaming → Low-latency audio generation ensures smooth back-and-forth conversations.
- Flexible Integration → Works seamlessly with AVR Core via AudioSocket.
- Private Agents Support → Use public ElevenLabs agents or your own private/customized ones with an API key.

## Configuration

### Environment Variables

To configure the ElevenLabs STS service, set the following variables:

Variable	Description	Example Value
PORT	Port on which the ElevenLabs STS service runs	6035
ELEVENLABS_AGENT_ID	Your ElevenLabs Agent ID (required)	your_agent_id
ELEVENLABS_API_KEY	API Key (only required for private agents)	sk-xxxx

## ⚠️⚠️⚠️ Audio Format Requirements 

Before using this integration, configure your ElevenLabs Agent with the following audio settings:

-	**TTS Output Format** → PCM 8000 Hz

![11labs-0.png](/images/elevenlabs/11labs-0.png)

- **User Input Audio Format** → PCM 8000 Hz


These settings are mandatory to ensure proper audio compatibility and real-time streaming performance with AVR.

### Example Docker Compose

```yaml
avr-sts-elevenlabs:
  image: agentvoiceresponse/avr-sts-elevenlabs
  platform: linux/x86_64
  container_name: avr-sts-elevenlabs
  restart: always
  environment:
    - PORT=6035
    - ELEVENLABS_AGENT_ID=$ELEVENLABS_AGENT_ID
    - ELEVENLABS_API_KEY=$ELEVENLABS_API_KEY
  networks:
    - avr

avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001
    - STS_URL=ws://avr-sts-elevenlabs:6035
  ports:
    - 5001:5001
  networks:
    - avr
```

## Reference

You can find the official repository here:
**Github**: [agentvoiceresponse/avr-sts-elevenlabs](https://github.com/agentvoiceresponse/avr-sts-elevenlabs)

For a complete example, check the [docker-compose-elevenlabs.yml](https://github.com/agentvoiceresponse/avr-infra/blob/main/docker-compose-elevenlabs.yml) in the [avr-infra](https://github.com/agentvoiceresponse/avr-infra) project.