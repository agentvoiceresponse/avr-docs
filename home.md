---
title: Overview
description: 
published: true
date: 2025-08-28T20:00:53.629Z
tags: 
editor: markdown
dateCreated: 2025-05-01T18:04:55.946Z
---

# Agent Voice Response (AVR)

Agent Voice Response (commonly abbreviated AVR) is a conversational AI platform that integrates with the open-source PBX Asterisk to replace traditional IVR (Interactive Voice Response) systems using AI-driven voice interactions. It acts as a voicebot, transcribing speech, generating responses through large language models (LLMs), and synthesizing speech back to callers.

## Overview

AVR offers a Docker-based deployment enabling organizations to integrate automatic speech recognition (ASR), large language models (LLM), text-to-speech (TTS), and direct speech-to-speech (STS) in voice calling systems. It supports a flexible, provider-agnostic architecture. Users can plug in services such as Google, Deepgram for ASR/TTS, and OpenAI or Anthropic for LLM, or opt for full voice-to-voice pipelines like OpenAI Realtime, Ultravox.ai, or Deepgram.

A key part of AVR’s design is its **intelligent audio handling**:
- **Voice Activity Detection (VAD)** ensures responsive and natural turn-taking during calls, allowing users to interrupt the agent smoothly.
- **Noise Handling & Ambient Sounds** enable background noise management for realistic or adaptive environments.
- **Speech-to-Speech (STS) Mode** delivers real-time voice-to-voice conversations without intermediate text processing.

## Features

- **Modular AI Components**: Swappable ASR, LLM, TTS, and STS blocks via HTTP APIs
- **Voice Activity Detection (VAD)**: Enables natural interruption and fast turn-taking
- **Noise & Ambient Sound Control**: Background audio simulation and noise reduction
- **Dockerized Microservices**: Each core service runs isolated in containers for scalability 
- **Support for Function-Calling**: Integration with OpenAI and Anthropic assistive features 
- **Speech-to-Speech Mode (STS)**: Direct voice transformation using providers like OpenAI Realtime, Ultravox, or Deepgram
- **Multi-language & Custom Voices**: Configurable voice personalities and multilingual support

## Architecture

- **Core Service** – Orchestrates voice input/output via Asterisk and AudioSocket
- **ASR Service** – Transcribes caller audio into text
- **LLM Service** – Generates conversational responses
- **TTS Service** – Converts responses back to speech
- **STS Service** – (Optional) Provides direct speech-to-speech mode for ultra-low latency
- **VAD & Noise Engine** – Handles voice activity detection and background noise simulation
- *Web GUI – (Under development) for deployment management*
- **Asterisk Integration** – Communication via SIP and AudioSocket

## Deployment

The `avr-infra` GitHub repository provides Docker Compose templates integrating AVR Core, Asterisk, and configurable ASR/LLM/TTS/STS stacks (e.g., Google, Deepgram, OpenAI, Anthropic, Vosk).  
Examples include combinations like **Deepgram + OpenAI**, **Google + OpenRouter**, **ElevenLabs + Anthropic**, and full voice pipelines via **OpenAI Realtime** or **Ultravox**.

## Use and Community

- **Installation**: Requires Docker and Asterisk (v18+ with AudioSocket); users register a SIP client and call a defined extension to interact with the AI agent
- **Licensing**: Distributed free for personal and commercial use; the AVR source remains proprietary though the infra and integration repos are open-source
- **Contributions**: Maintainers host a Discord community and accept donations via Ko-fi

## GitHub Repositories

Some notable public repositories under the `agentvoiceresponse` organization include:
- *avr-infra* – Docker Compose templates for deploying AVR with Asterisk and AI services
- *avr-llm-openai, avr-llm-openai-assistant* – OpenAI-based LLM connectors
- *avr-asr-google-cloud-speech, avr-asr-vosk* – ASR modules leveraging Google Speech or open-source Vosk
- *avr-tts-google-speech-tts, avr-tts-openai, avr-tts-coquitts* – TTS connectors for various engines
- *avr-sts-openai, avr-sts-ultravox, avr-sts-deepgram* – STS connectors for various providers
- *avr-vad* – VAD library (Silero) for speech activity detection, available as an npm package
- *avr-ambient-noise* – Background noise and ambient sound management
- *avr-asterisk* – Docker image of Asterisk optimized for AVR environments