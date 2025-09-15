---
title: Coqui TTS
description: 
published: true
date: 2025-09-13T14:55:00.894Z
tags: 
editor: markdown
dateCreated: 2025-09-13T14:54:56.595Z
---

# Coqui TTS Integration

**Coqui TTS** is an open-source, high-quality **text-to-speech engine** built on deep learning.  
It provides natural-sounding voices and supports multiple languages, making it a great option for **local deployments** where flexibility and privacy are priorities.  

AVR integrates with CoquiTTS to enable real-time speech synthesis inside your telephony infrastructure.

## Advantages of Using CoquiTTS

- **Open Source** → No vendor lock-in, free to customize  
- **Local Deployment** → Runs fully on your own hardware, ideal for privacy-sensitive use cases  
- **Multi-language Support** → Large selection of pre-trained voice models available  
- **Flexible Models** → Choose from different architectures (VITS, FastPitch, Tacotron2, etc.)  
- **Custom Voices** → Train your own voices with open datasets  

## Configuration

### Environment Variables

| Variable | Description                          | Example Value |
|----------|--------------------------------------|---------------|
| `PORT`   | Port where the CoquiTTS service runs | `6032`        |

Example `.env` file:
```bash
PORT=6032
```

### Docker Compose Example

Below is a sample configuration using AVR with CoquiTTS:

```yaml
avr-tts-coquitts:
  image: agentvoiceresponse/avr-tts-coquitts
  platform: linux/x86_64
  container_name: avr-tts-coquitts
  restart: always
  environment:
    - PORT=6032
  ports:
    - 6032:6032
  networks:
    - avr

avr-coqui-ai-tts:
  image: ghcr.io/coqui-ai/tts-cpu
  platform: linux/x86_64
  container_name: avr-coqui-ai-tts
  entrypoint: "python3 TTS/server/server.py --model_name tts_models/en/vctk/vits"
  restart: always
  environment:
    - PORT=5002
  ports:
    - 5002:5002
  networks:
    - avr
```

**Notes**:
- avr-tts-coquitts acts as the AVR microservice wrapper.
- avr-coqui-ai-tts runs the official Coqui TTS server with a selected model.
- You can replace tts_models/en/vctk/vits with any supported model from Coqui TTS models.

### How It Works

1.	AVR sends text requests to avr-tts-coquitts.
2.	The request is forwarded to the running Coqui TTS server (avr-coqui-ai-tts).
3.	Coqui synthesizes speech using the selected model.
4.	The output is downsampled to 8kHz PCM for compatibility with Asterisk AudioSocket.
5.	AVR streams audio back to the caller in 320-byte chunks for real-time playback.

### Example Test with cURL

```bash
curl -X POST http://localhost:6032/text-to-speech-stream \
     -H "Content-Type: application/json" \
     -d '{"text": "Hello, this is CoquiTTS running with Agent Voice Response!"}' \
     --output response.raw
```

The resulting response.raw file will contain PCM audio at 8kHz.

### Best Practices

- Test different Coqui models (tts_models/...) to find the best trade-off between latency and voice quality.
- Run with GPU acceleration for production or real-time scenarios.
- Keep your models directory mounted as a Docker volume to avoid re-downloading on container restarts.

## Community & Support

- GitHub: [avr-tts-coquitts](https://github.com/agentvoiceresponse/avr-tts-coquitts)