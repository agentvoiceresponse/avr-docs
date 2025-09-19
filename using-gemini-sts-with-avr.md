---
title: Using Gemini STS with AVR
description: 
published: true
date: 2025-09-19T15:49:50.029Z
tags: 
editor: markdown
dateCreated: 2025-09-02T13:38:04.526Z
---

# Using Gemini STS with AVR

<div align="center">
  <img src="/images/gemini/gemini-logo.png" alt="Gemini Logo" width="300"/>
</div>

Gemini is Google’s family of multimodal AI models. With the Gemini STS (Speech-to-Speech) integration, AVR can natively handle conversations where speech in is directly transformed into speech out—without requiring separate ASR (speech-to-text) and TTS (text-to-speech) components.

This reduces latency and delivers more natural, human-like dialogue.

## Why use Gemini STS?
- End-to-End Speech Conversations — Direct speech-to-speech transformation, bypassing intermediate text steps.
- Low Latency — Faster interactions compared to ASR + LLM + TTS pipelines.
- Multilingual Support — Gemini supports multiple languages out of the box.
- Instruction-Tuned — You can set custom instructions for your voice assistant (e.g., “You are a helpful assistant.”).
- Cloud-Native — Runs on Google’s infrastructure, ensuring scalability and performance.

## Generate API Credentials

To connect AVR with Gemini, you need an API Key:

1.	Open Google AI Studio.
2.	Sign in with your Google account.
3.	Navigate to API Keys and create a new key.
4.	Copy the key and store it securely.

You’ll use this key as `GEMINI_API_KEY` in your Docker environment.

## Repository
- Github: [avr-sts-gemini](https://github.com/agentvoiceresponse/avr-sts-gemini)

## Environment Variables

| Variable             | Description                             | Example Value                                  |
|----------------------|-----------------------------------------|-----------------------------------------------|
| `PORT`               | Port on which the Gemini STS service runs | `6037`                                        |
| `GEMINI_API_KEY`     | API Key from Google AI Studio           | `AIza...`                                     |
| `GEMINI_MODEL`       | Gemini model ID to use                  | `gemini-2.5-flash-preview-native-audio-dialog`|
| `GEMINI_INSTRUCTIONS`| System prompt for the voice assistant   | `"You are a helpful assistant."`              |

## Docker Setup

Add the following service to your docker-compose.yml:

```yaml
avr-sts-gemini:
  image: agentvoiceresponse/avr-sts-gemini
  platform: linux/x86_64
  container_name: avr-sts-gemini
  restart: always
  environment:
    - PORT=6037
    - GEMINI_API_KEY=$GEMINI_API_KEY
    - GEMINI_MODEL=$GEMINI_MODEL
    - GEMINI_INSTRUCTIONS=$GEMINI_INSTRUCTIONS
  networks:
    - avr
```

## Integration with AVR Core

Point avr-core to the Gemini STS service:

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001 
    - STS_URL=ws://avr-sts-gemini:6037
  ports:
    - 5001:5001
  networks:
    - avr
```

## Example in AVR Infra

A ready-to-use integration example is available in the [avr-infra](https://github.com/agentvoiceresponse/avr-infra) github project:

`docker-compose-gemini.yml` — Example 10 configuration of AVR with Gemini STS.

## Performance Notes

- Best for real-time conversations due to low latency.
- Requires a stable internet connection (cloud-based API).
- Great option if you don’t want to manage separate ASR + LLM + TTS services.