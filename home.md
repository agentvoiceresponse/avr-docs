---
title: Overview
description: 
published: true
date: 2025-08-08T15:21:04.641Z
tags: 
editor: markdown
dateCreated: 2025-05-01T18:04:55.946Z
---

# Agent Voice Response (AVR)

Agent Voice Response (commonly abbreviated AVR) is a conversational AI platform that integrates with the open‑source PBX Asterisk to replace traditional IVR (Interactive Voice Response) systems using AI-driven voice interactions. It acts as a voicebot, transcribing speech, generating responses through large language models (LLMs), and synthesizing speech back to callers.

## Overview

AVR offers a Docker-based deployment enabling organizations to integrate automatic speech recognition (ASR), LLM, text-to-speech (TTS), and direct speech-to-speech (STS) in voice calling systems. It supports a flexible, provider-agnostic architecture. Users can plug in services such as Google, Deepgram for ASR/TTS, and OpenAI or Anthropic for LLM, or opt for full voice‑to‑voice pipelines like OpenAI Realtime or Ultravox.ai.

## Features

- **Modular AI Components**: Swappable ASR, LLM, and TTS Blocks via HTTP APIs
- **Voice Activity Detection (VAD)**: Enables natural interruption in voice calls
- **Dockerized Microservices**: Each core service runs isolated in containers for scalability 
- **Support for Function‑Calling**: Integration with OpenAI and Anthropic assistive features 
- **Speech‑to‑Speech Mode**: Direct voice transformation using providers like OpenAI Realtime
- **Multi‑language & Custom Voices**: Offerings include configurable voice personalities and language support

## Architecture

- **Core Service** – Orchestrates voice input/output via Asterisk and AudioSocket
- **ASR Service** – Transcribes caller audio into text
- **LLM Service** – Generates conversational responses
- **TTS Service** – Converts responses back to speech
- *Web GUI – (Under development) for deployment management*
- **Asterisk Integration** – Communication via SIP and AudioSocket

## Deployment

The avr-infra GitHub repository provides Docker Compose templates integrating AVR Core, Asterisk, and configurable ASR/LLM/TTS stacks (e.g., Google, Deepgram, OpenAI, Anthropic, Vosk).
Examples include combinations like Deepgram + OpenAI, Google + OpenRouter, ElevenLabs + Anthropic, and full voice pipelines via OpenAI Realtime or Ultravox.

## Use and Community

- **Installation**: Requires Docker and Asterisk (v18+ with AudioSocket); users register a SIP client and call a defined extension to interact with the AI agent
- **Licensing**: Distributed free for personal and commercial use; the AVR source remains proprietary though the infra and integration repos are open‑source
- **Contributions**: Maintainers host a Discord community and accept donations via Ko‑fi

## GitHub Repositories

Some notable public repositories under the agentvoiceresponse organization include:
- *avr-infra* – Docker Compose templates for deploying AVR with Asterisk and AI services
- *avr-llm-openai, avr-llm-openai-assistant* – OpenAI-based LLM connectors
- *avr-asr-google-cloud-speech, avr-asr-vosk* – ASR modules leveraging Google Speech or open-source Vosk
- *avr-tts-google-speech-tts, avr-tts-openai, avr-tts-coquitts* – TTS connectors for various engines
- *avr-sts-openai, avr-sts-ultravox* - STS connectors for varous engines
- *avr-vad* – VAD library (Silero) for speech activity detection, available as an npm package
- *avr-asterisk* – Docker image of Asterisk optimized for AVR environments