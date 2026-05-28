---
title: Using Gemini STS with AVR
description: 
published: true
date: 2026-05-28T20:45:00.000Z
tags: 
editor: markdown
dateCreated: 2025-09-02T13:38:04.526Z
---

# Using Gemini STS with AVR

<div align="center">
  <img src="/images/gemini/gemini-logo.png" alt="Gemini Logo" width="300"/>
</div>

**Gemini** is Google’s family of multimodal AI models. With the **Gemini Speech-to-Speech (STS)** integration, Agent Voice Response (AVR) can handle conversations where **speech input is directly transformed into speech output**, without requiring separate ASR (speech-to-text) and TTS (text-to-speech) components.

This approach significantly **reduces latency** and enables more **natural, human-like voice interactions**.



## Why Use Gemini STS?

- **End-to-End Speech Conversations** — Direct speech-to-speech transformation.
- **Low Latency** — Faster than traditional ASR + LLM + TTS pipelines.
- **Multilingual Support** — Supports multiple languages out of the box.
- **Instruction-Tuned** — Fully customizable system prompts.
- **Cloud-Native** — Runs on Google’s scalable and reliable infrastructure.



## Authentication

`avr-sts-gemini` **1.5.0+** supports two ways to authenticate with Gemini Live. Choose **one** mode per deployment — do not set an API key and Vertex flags together unless you intend Vertex mode (Vertex takes precedence when `GOOGLE_GENAI_USE_VERTEXAI=true`).

| Mode | Best for | Credentials |
|------|----------|-------------|
| **Google AI Studio** (default) | Quick setup, local dev | API key (`GEMINI_API_KEY` or `GOOGLE_API_KEY`) |
| **Vertex AI** (Google Cloud Console) | GCP projects, service accounts, enterprise | Application Default Credentials (ADC) + project/region |

Connector reference: https://github.com/agentvoiceresponse/avr-sts-gemini



### Google AI Studio (API key)

1. Open **Google AI Studio**: https://aistudio.google.com/apikey
2. Sign in with your Google account
3. Create an API key and copy it securely
4. Set `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) in your environment

No Google Cloud project is required for this path.



### Vertex AI (Google Cloud Console)

Use Vertex when Gemini is enabled in [Google Cloud Console](https://console.cloud.google.com/) instead of AI Studio.

1. Create or select a **GCP project** with billing enabled
2. Enable the **[Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)**
3. Choose a **region** (for example `us-central1`) — set `GOOGLE_CLOUD_LOCATION` to match
4. Configure **Application Default Credentials** using one of:
   - **Service account (recommended for Docker):** create a key JSON file and set `GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json` (mount the file into the container)
   - **Local development:** run `gcloud auth application-default login`
5. Set Vertex mode environment variables (see tables below)

Optional AVR-prefixed aliases: `GEMINI_USE_VERTEXAI`, `GEMINI_VERTEX_PROJECT`, `GEMINI_VERTEX_LOCATION`.



## Repository

- GitHub: https://github.com/agentvoiceresponse/avr-sts-gemini



## Environment Variables

### Shared (both modes)

| Variable | Description | Example Value |
|--------|-------------|---------------|
| `PORT` | Port on which the Gemini STS service runs | `6037` |
| `GEMINI_MODEL` | Gemini model ID (Live native audio) | `gemini-2.5-flash-native-audio-preview-12-2025` |
| `GEMINI_API_VERSION` | Optional API version override | *(unset)* |
| `GEMINI_INSTRUCTIONS` | System prompt for the voice assistant | `"You are a helpful assistant."` |
| `GEMINI_URL_INSTRUCTIONS` | URL to fetch dynamic instructions | `https://your-api.com/instructions` |
| `GEMINI_FILE_INSTRUCTIONS` | Path to local instruction file | `./instructions.txt` |

### Google AI Studio only

| Variable | Required | Description | Example Value |
|--------|----------|-------------|---------------|
| `GEMINI_API_KEY` | Yes* | API key from Google AI Studio | `AIza...` |
| `GOOGLE_API_KEY` | Yes* | Alternative name (Google GenAI SDK) | `AIza...` |

\* One of `GEMINI_API_KEY` or `GOOGLE_API_KEY` is required. Do **not** set `GOOGLE_GENAI_USE_VERTEXAI=true` for AI Studio mode.

### Vertex AI only

| Variable | Required | Description | Example Value |
|--------|----------|-------------|---------------|
| `GOOGLE_GENAI_USE_VERTEXAI` | Yes | Enable Vertex AI mode | `true` |
| `GOOGLE_CLOUD_PROJECT` | Yes | GCP project ID | `my-avr-project` |
| `GOOGLE_CLOUD_LOCATION` | Yes | Vertex region | `us-central1` |
| `GOOGLE_APPLICATION_CREDENTIALS` | Yes† | Path to service account JSON (in container) | `/secrets/gcp-sa.json` |

† Required in Docker/production unless the host provides ADC another way (e.g. GCE metadata).

Aliases: `GEMINI_USE_VERTEXAI`, `GEMINI_VERTEX_PROJECT`, `GEMINI_VERTEX_LOCATION`.

### Gemini thinking settings

We’ve added support for the following Gemini settings:

- `GEMINI_THINKING_LEVEL=MINIMAL`
- `GEMINI_THINKING_BUDGET=0`

More details here 👉 https://ai.google.dev/gemini-api/docs/thinking?hl=en

Supported values for `GEMINI_THINKING_LEVEL`:

- `THINKING_LEVEL_UNSPECIFIED`
- `LOW`
- `MEDIUM`
- `HIGH`
- `MINIMAL`

`GEMINI_THINKING_BUDGET`:

- `0` → turn off thinking
- `-1` → enable dynamic thinking


## Docker Setup

Pin the image tag in production (for example `1.5.0`). `:latest` tracks the newest release.

### Google AI Studio

Add the following service to your `docker-compose.yml`:

```yaml
avr-sts-gemini:
  image: agentvoiceresponse/avr-sts-gemini:1.5.0
  platform: linux/x86_64
  container_name: avr-sts-gemini
  restart: always
  environment:
    - PORT=6037
    - GEMINI_API_KEY=${GEMINI_API_KEY}
    - GEMINI_MODEL=${GEMINI_MODEL:-gemini-2.5-flash-native-audio-preview-12-2025}
    - GEMINI_INSTRUCTIONS=${GEMINI_INSTRUCTIONS:-You are a helpful assistant}
    - GEMINI_THINKING_LEVEL=${GEMINI_THINKING_LEVEL:-MINIMAL}
    - GEMINI_THINKING_BUDGET=${GEMINI_THINKING_BUDGET:-0}
  networks:
    - avr
```



### Vertex AI

Mount a service account key and enable Vertex mode:

```yaml
avr-sts-gemini:
  image: agentvoiceresponse/avr-sts-gemini:1.5.0
  platform: linux/x86_64
  container_name: avr-sts-gemini
  restart: always
  environment:
    - PORT=6037
    - GOOGLE_GENAI_USE_VERTEXAI=true
    - GOOGLE_CLOUD_PROJECT=${GOOGLE_CLOUD_PROJECT}
    - GOOGLE_CLOUD_LOCATION=${GOOGLE_CLOUD_LOCATION:-us-central1}
    - GOOGLE_APPLICATION_CREDENTIALS=/run/secrets/gcp-sa.json
    - GEMINI_MODEL=${GEMINI_MODEL:-gemini-2.5-flash-native-audio-preview-12-2025}
    - GEMINI_INSTRUCTIONS=${GEMINI_INSTRUCTIONS:-You are a helpful assistant}
  volumes:
    - ${GOOGLE_APPLICATION_CREDENTIALS_HOST_PATH}:/run/secrets/gcp-sa.json:ro
  networks:
    - avr
```

Set `GOOGLE_APPLICATION_CREDENTIALS_HOST_PATH` in your `.env` to the JSON key path on the host (for example `./secrets/gcp-sa.json`).

> If WebSocket init fails, the connector returns a specific `error` message (missing API key, project, or location) instead of a generic failure — check container logs and the message from `avr-core`.



## Integration with AVR Core

Configure **avr-core** to use Gemini STS by setting `STS_URL`:

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

> ⚠️ When `STS_URL` is configured, `ASR_URL`, `LLM_URL`, and `TTS_URL` must be commented out.



## Instruction Loading Methods

The Gemini STS integration supports **multiple instruction sources** with a clear priority order.

### 1. Environment Variable (Highest Priority)

```env
GEMINI_INSTRUCTIONS="You are a specialized customer service agent for a tech company. Always be polite and helpful."
```

If set, this overrides all other instruction sources.



### 2. Web Service (Medium Priority)

```env
GEMINI_URL_INSTRUCTIONS="https://your-api.com/instructions"
```

Expected response format:

```json
{
  "system": "You are a helpful assistant that provides technical support."
}
```

The request will include the call UUID as an HTTP header:

```
X-AVR-UUID: <call-uuid>
```

This enables **dynamic, per-call instruction generation**.



### 3. File-Based Instructions (Lowest Priority)

```env
GEMINI_FILE_INSTRUCTIONS="./instructions.txt"
```

The file should contain plain text instructions.



### Instruction Priority Order

1. `GEMINI_INSTRUCTIONS`
2. `GEMINI_URL_INSTRUCTIONS`
3. `GEMINI_FILE_INSTRUCTIONS`
4. Default instructions (fallback)



## Example in AVR Infra

A ready-to-use example is available in the **avr-infra** repository:

- `docker-compose-gemini.yml` — Example #10

https://github.com/agentvoiceresponse/avr-infra



## Performance Notes

- Optimized for real-time, low-latency conversations
- Requires a stable internet connection
- VAD is handled natively by Gemini
- AVR Core VAD and `INTERRUPT_LISTENING` are ignored in STS mode