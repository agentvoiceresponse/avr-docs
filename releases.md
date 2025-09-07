---
title: Release Notes
description: List of new features, bug fixes and improvement
published: true
date: 2025-09-07T12:48:40.590Z
tags: 
editor: markdown
dateCreated: 2025-08-08T16:52:47.453Z
---

> 6 September 2025
> {.is-info}
{.is-info}

We’re releasing a new way to load prompt instructions (only avr-sts-openai).

As AVR projects get more and more complex, we decided to implement a dynamic instruction loading mode. Here’s how it works:

# Instruction Loading Methods

The application supports three different methods for loading AI instructions, with a specific priority order:

**1. Environment Variable (Highest Priority)**
Set the OPENAI_INSTRUCTIONS environment variable with your custom instructions:

```
OPENAI_INSTRUCTIONS="You are a specialized customer service agent for a tech company. Always be polite and helpful."
```
**2. Web Service (Medium Priority)**
If no environment variable is set, the app can fetch instructions from a web service using the OPENAI_URL_INSTRUCTIONS environment variable:

```
OPENAI_URL_INSTRUCTIONS="https://your-api.com/instructions"
```

The web service should return a JSON response with a system field containing the instructions:

```json
{
  "system": "You are a helpful assistant that provides technical support."
}
```
The application will include the session UUID in the request headers as X-AVR-UUID for personalized instructions.

**3. File (Lowest Priority)**
If neither env var nor web service is configured, the app can load instructions from a local file using OPENAI_FILE_INSTRUCTIONS:

```
OPENAI_FILE_INSTRUCTIONS="./instructions.txt"
```

The file should contain plain text instructions that will be used as the system prompt.

--- 

📖 Updated docs: https://wiki.agentvoiceresponse.com/en/using-openai-realtime-sts-with-avr


> 6 September 2025
> {.is-info}
{.is-info}

Lots of updates to share, so buckle up 🎉

🔹 Spoiler revealed!
We’ve just released the very first avr-sts-elevenlabs integration 🗣️➡️🗣️
This means you can now leverage ElevenLabs for speech-to-speech with:
    •    Much more natural and realistic voices
    •    Lower latency in voice generation
    •    A wide variety of expressive voices and styles

👉 How many of you have been waiting for this? 😎
More details here: https://wiki.agentvoiceresponse.com/en/elevenlabs-speech-to-speech-integration-avr

🔹 But that’s not all!
This week we’re also rolling out updates for avr-sts-openai and avr-sts-gemini with full tools support 🛠️
I’ve prepared a guide on how to use tools and even implement your own custom function calls:
https://wiki.agentvoiceresponse.com/en/avr-function-calls

🔹 Core improvements 💥 💥 💥 💥 💥 💥 💥 
We noticed some latency with HTTP streaming between avr-core and avr-sts, so we rewrote the communication layer using WebSockets. 😅 
This brings:
    •    Zero latency audio streaming
    •    Support for user/agent interruptions via WebSocket events

Updated docs on events here:
https://wiki.agentvoiceresponse.com/en/avr-sts-integration-implementation

⚙️ In your avr-core Docker, update:
- STS_URL=ws://avr-sts-openai:6030

instead of
- STS_URL=http://avr-sts-openai:6030/speech-to-speech-stream


No worries—HTTP stream is still backward compatible, but we strongly recommend switching to WebSockets ASAP.
Official integrations (OpenAI, Ultravox, Deepgram, ElevenLabs, Gemini) are already converted. HTTP stream code will no longer be maintained.

🔹 Infra update
We’ve refreshed all URLs in avr-infra—Docker Compose examples are already updated.

The result? Conversations are now way more natural and seamless. We’re really happy with how it feels. 
Oh, almost forgot—docs have also been updated:
    •    https://wiki.agentvoiceresponse.com/en/using-openai-realtime-sts-with-avr
    •    https://wiki.agentvoiceresponse.com/en/using-gemini-sts-with-avr

> 31 August 2025
> {.is-info}
{.is-info}

We just released a new version of avr-sts-openai 1.4.0! 🚀

In this release, We’ve improved the downsample and upsample handling using a library we found online: @alexanderolsen/libsamplerate-js. The audio quality seems much better now, but we’d love to hear your thoughts.

---

We are super excited to announce the release of avr-sts-gemini v1.0.0 https://github.com/agentvoiceresponse/avr-sts-gemini 
From now on, you can integrate AgentVoiceResponse directly with Gemini! ✨

👉 What’s new and why it’s awesome?
    •    Native speech-to-speech integration with Gemini.
    •    Faster and more natural conversations powered by Gemini’s models.
    •    Even more flexibility to choose the AI provider that best fits your use case.

📌 I’ve also added a brand-new example (Example 10) to the avr-infra project:
🔗 https://github.com/agentvoiceresponse/avr-infra?tab=readme-ov-file#example-10-gemini-speech-to-speech

📖 Full documentation is available here:
🔗 https://wiki.agentvoiceresponse.com/en/using-gemini-sts-with-avr

> 24 August 2025
> {.is-info}
{.is-info}

We’ve just released a new version of avr-core (1.6.0) with support for **ambient background noise** 🎉
You can find the documentation on how to use it here:
👉 [using-ambient-background-noise-in-avr](/using-ambient-background-noise-in-avr)

With this feature, AVR can simulate more realistic environments, making your tests and demos feel much closer to real-world conditions.

At the same time, We’ve also released avr-infra (1.3.0), which now includes an ambient_sounds directory. For now, it contains one sample file: office_background.raw (RAW format).
In the documentation, you’ll also find an example using SoX to convert WAV to RAW, but you can also use other tools like ffmpeg, etc.

Here’s an example of how to configure it in your docker-compose.yml:
```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001 
    - STS_URL=http://avr-sts-deepgram:6033/speech-to-speech-stream
    - AMBIENT_NOISE_FILE=ambient_sounds/office_background.raw
    - AMBIENT_NOISE_LEVEL=0.90
  volumes:
    - ./ambient_sounds:/usr/src/app/ambient_sounds
  ports:
    - 5001:5001
  networks:
    - avr
```

We’ve also released the first version of avr-llm-n8n (1.0.0) 🎉 → https://github.com/agentvoiceresponse/avr-llm-n8n

<div align="center">
  <img src="/images/n8n/n8n.png" alt="FreePBX" width="300"/>
</div>

Integrating AVR with n8n allows you to build AI-powered voicebots with visual workflows and direct integration with AVR.

I’ve also published a docker-compose example with Deepgram as ASR & TTS and n8n as LLM here:
👉 https://github.com/agentvoiceresponse/avr-infra/blob/main/docker-compose-n8n.yml

And here you can find the official documentation, including a basic example of an AI Agent configured on n8n:
👉 https://wiki.agentvoiceresponse.com/en/using-avr-with-n8n

You can of course use your own n8n, but if you don’t have a local or cloud instance, the docker-compose I shared also includes an n8n service that will be installed together with the other containers.

Have fun! 🚀 I’ve already tested some cool integrations with Google Calendar, Google Sheets, CRM systems, etc. — and they work really well.

Thanks in advance to anyone who wants to share their own workflow! We could even create a dedicated wiki section for community use cases.

> 22 August 2025
> {.is-info}
{.is-info}

📢 Update on AVR-STS-ULTRAVOX integration

The avr-sts-ultravox integration now supports not only creating calls with a configurable Agent ID (from the UI) but also creating a generic call via prompt 🎉.

In the README you’ll find details on how to configure both modes:
    •    Agent mode (default, nothing has changed here)
    •    Generic mode (enabled by setting *ULTRAVOX_CALL_TYPE=generic*)

When using generic mode, you’ll need to configure the following variables:
```
ULTRAVOX_SYSTEM_PROMPT: System prompt for the AI (default: "You are a helpful AI assistant.")
ULTRAVOX_TEMPERATURE: AI temperature setting (default: 0)
ULTRAVOX_MODEL: AI model to use (default: "fixie-ai/ultravox")
ULTRAVOX_VOICE: Voice to use (default: "alloy")
ULTRAVOX_RECORDING_ENABLED: Enable call recording (default: false)
ULTRAVOX_JOIN_TIMEOUT: Join timeout (default: "30s")
ULTRAVOX_MAX_DURATION: Maximum call duration (default: "3600s")
```
🙏 Thanks everyone for the support and contributions — more updates coming soon!

> 13 August 2025
> {.is-info}
{.is-info}

:rocket: Big thanks to @mirkuz93  for contributing the Deepgram Speech-to-Speech integration for AVR — avr-sts-deepgram! :tada:
Code is here: https://github.com/agentvoiceresponse/avr-sts-deepgram
Docker Compose example: Example 8 – Deepgram Speech-to-Speech https://github.com/agentvoiceresponse/avr-infra?tab=readme-ov-file#example-8-deepgram-speech-to-speech

:bulb: Super easy to use:
:one: Generate your Deepgram API key (or reuse the one from your ASR/TTS integration).
:two: Configure your .env file:
```
PORT=6033
DEEPGRAM_API_KEY=
AGENT_PROMPT=
```
:gear: Extra config for ASR, LLM & TTS models:
```
DEEPGRAM_SAMPLE_RATE=   # default: 8000
DEEPGRAM_ASR_MODEL=     # default: nova-3
DEEPGRAM_TTS_MODEL=     # default: aura-2-thalia-en
DEEPGRAM_GREETING=
OPENAI_MODEL=           # default: gpt-4o-mini
```

> 8 August 2025
> {.is-info}
{.is-info}


🚀 New AVR-Core Release: v1.5.6 is out!

We’ve just released a new version of avr-core (v1.5.5) with a small but powerful update:
🔹 Every HTTP request now includes a X-UUID header!
You can retrieve it easily from your **STS** and **ASR** services like this:
```
const handleAudioStream = async (req, res) => {
  const uuid = req.headers['x-uuid'];
  console.log('Received UUID:', uuid);
...
```
⚙️ I’ve also released a new version of avr-sts-ultravox (v.1.1.1), which now grabs the UUID and injects it into the metadata (see attach). This means… you can now use it directly inside your tools!

🧩 Feel free to share any cool tool examples with the community on #avr-self-hosting — we’d love to see what you build.

Have fun and happy hacking! 🎧🧠✨
Thanks @everyone  and see you soon! 👋

> 3 August 2025
{.is-info}


🚀 New Release: STS Ultravox.ai & STS OpenAI Realtime 🛠️
Hi @everyone !
I’ve just released a new version of STS Ultravox.ai with significantly improved audio quality. I’d love to hear your feedback!

I’ve also published an update to STS OpenAI Realtime, which now includes a VAD-based interruption mechanism 🧠🎙️. Let me know how it works for you!

Have a great day and happy building!

> 2 August 2025
{.is-info}


🔥 Big Update Today! 🔥

Hey @everyone ! Hope you’re all doing great — Today I’m dropping a bunch of cool stuff for you to try out 👇


---


🤖 AVR - STS now works with **Ultravox.ai**!

Yup, it’s here! First working integration between AVR and **Ultravox.ai** 🎯
🛠️ GitHub repo:
https://github.com/agentvoiceresponse/avr-sts-ultravox

🧪 Test it right away with a ready-made Docker setup:
https://github.com/agentvoiceresponse/avr-infra → docker-compose-ultravox.yml

🔑 Quick start:
    1.    Sign up on https://www.ultravox.ai
    2.    Create your agent (set language, welcome message, etc.)
    3.    Generate your API key
    4.    Edit your .env:

ULTRAVOX_AGENT_ID=your_agent_id  
ULTRAVOX_API_KEY=your_api_key

✅ That’s it — your voice agent is live!


---


🧠 New: Audio Resampler Library!

To make all this smoother, I’ve built a brand new audio resampling library — super light, reusable across projects, and optimized for STS!

🔗 GitHub:
https://github.com/agentvoiceresponse/avr-resampler

📦 NPM:
https://www.npmjs.com/package/avr-resampler

Check out the README for setup and examples. It’s already being used in production!


---


🎧 Better audio in avr-sts-openai

I’ve also updated avr-sts-openai with the new resampler.
💡 Audio is cleaner, tighter — still room for improvement, but it’s a nice step forward.
I’ll keep tweaking it in the next few days!


---


💬 As always, I’d love your feedback, suggestions, bug reports, crazy ideas — bring it all.
Thank you for being part of this 🚀
Let’s keep building!

> 13 June 2025
{.is-info}


🧠 avr-vad v1.0-7
    •    Fix: Improved VAD (Voice Activity Detection) compatibility with 8000Hz 16bit PCM audio. Detection is now more accurate and stable.

---
 avr-core v1.5.0
    1.    VAD Integration
We’ve integrated the avr-vad library into avr-core. This allows instant interruption of the Agent AI when speech is detected.
➤ Be sure to set:

INTERRUPT_LISTENING=false

This ensures ASR keeps listening, even during user interruptions.
Default VAD parameters:
```env
positiveSpeechThreshold: process.env.VAD_POSITIVE_SPEECH_THRESHOLD || 0.08,
negativeSpeechThreshold: process.env.VAD_NEGATIVE_SPEECH_THRESHOLD || 0.03,
minSpeechFrames: process.env.VAD_MIN_SPEECH_FRAMES || 3,
preSpeechPadFrames: process.env.VAD_PRE_SPEECH_PAD_FRAMES || 3,
redemptionFrames: process.env.VAD_REDEMPTION_FRAMES || 8,
frameSamples: process.env.VAD_FRAME_SAMPLES || 512,
sampleRate: 8000,
model: process.env.VAD_MODEL || "v5"
```
More info in the avr-vad README.

2. AudioSocket Dial Support

You can now scale your AVR project to handle multiple channels using the native Dial() app with AudioSocket:

```env
exten => 5001,1,Answer()
exten => 5001,n,Ringing()
exten => 5001,n,Wait(1)
exten => 5001,n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
exten => 5001,n,Dial(AudioSocket/avr-core:5001/${UUID})
exten => 5001,n,Hangup()
```

3. Assistant Text Cleanup Fixes
	• In handleText, we now clean 【】 tags across chunk boundaries before sentence splitting.
  • In cleanText, the 【】 cleanup was removed, as it’s now handled upstream.

→ This ensures tags like "【document.t" + "xt】" are correctly removed once fully received.

---

avr-infra v1.2.0
    •    Update: Substituted the legacy AudioSocket application with the new Dial(AudioSocket) syntax.
→ More aligned with avr-core 1.5.0 and scalable setups.





