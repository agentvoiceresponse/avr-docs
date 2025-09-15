---
title: Using Vosk ASR with AVR
description: 
published: true
date: 2025-08-31T20:00:44.941Z
tags: 
editor: markdown
dateCreated: 2025-08-31T20:00:40.321Z
---

# Using Vosk ASR with AVR

Vosk is an open-source speech recognition toolkit that runs locally without requiring internet connectivity. It supports more than 20 languages and dialects and is a great choice if you need offline, privacy-friendly transcription.

## Supported Languages

Vosk provides pre-trained models in over 20 languages, including:  
English, Indian English, German, French, Spanish, Portuguese, Chinese, Russian, Turkish, Vietnamese, Italian, Dutch, Catalan, Arabic, Greek, Farsi, Filipino, Ukrainian, Kazakh, Swedish, Japanese, Esperanto, Hindi, Czech, Polish, Uzbek, Korean, Breton, Gujarati, Tajik, Telugu.  

**Full list and downloads available here**: [Vosk Models](https://alphacephei.com/vosk/models)

## Docker Setup

To use Vosk with AVR, add the following service to your `docker-compose.yml`:

```yaml
avr-asr-vosk:
  image: agentvoiceresponse/avr-asr-vosk
  platform: linux/x86_64
  container_name: avr-asr-vosk
  restart: always
  environment:
    - PORT=6010
    - MODEL_PATH=model
  volumes:
    - ./model:/usr/src/app/model
  networks:
    - avr
```

## Example: AVR Core + Vosk

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  container_name: avr-core
  restart: always
  environment:
    - ASR_URL=http://avr-asr-vosk:6010/speech-to-text-stream
    ...
  networks:
    - avr
```

## Environment Variables

Variable
Description
Example Value
PORT
Port on which the Vosk ASR service runs
6010
MODEL_PATH
Path to the mounted Vosk model (container)
model

## Performance Notes
- **Small models** → Faster, require fewer resources, but less accurate
- **Large models** → Higher accuracy, but demand more CPU/RAM/GPU

For most AVR use cases, small/medium models provide a good balance between latency and accuracy

## Troubleshooting
- **Error: No model found** → Ensure the model directory is correctly mounted and not empty
- **High CPU usage** → Switch to a smaller model or allocate more resources
- **Slow recognition** → Verify Docker host has sufficient CPU cores and RAM

