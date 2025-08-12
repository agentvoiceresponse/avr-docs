---
title: Release Notes
description: List of new features, bug fixes and improvement
published: true
date: 2025-08-12T10:20:32.242Z
tags: 
editor: markdown
dateCreated: 2025-08-08T16:52:47.453Z
---

> 12 August 2025
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

> 6 August 2025
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

> 11 June 2025
{.is-info}



🚀 Changelog Update – June 11, 2025
Hi @everyone! Here’s a quick summary of what’s new today across the AVR ecosystem:

---

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
```
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

```
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





