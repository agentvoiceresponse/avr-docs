---
title: Get Started
description: 
published: true
date: 2025-08-06T17:10:55.152Z
tags: 
editor: markdown
dateCreated: 2025-08-06T17:06:33.271Z
---

# Get Started with Agent Voice Response (AVR)

## Overview

The AVR Infrastructure project provides a complete, modular deployment environment for the Agent Voice Response system. It allows you to launch the AVR Core, ASR, LLM, and TTS services, all integrated with an Asterisk PBX using the AudioSocket protocol.

This setup supports a wide range of providers—including OpenAI, Deepgram, Google, ElevenLabs, Vosk, and Anthropic—and can be customized with Docker Compose.

## Prerequisites

Before starting, ensure the following tools and credentials are ready:
- Docker and Docker Compose installed
- Valid API keys for services you plan to use (e.g., OpenAI, Google, Deepgram)
- (Optional) SIP client installed (e.g., Zoiper, Linphone) for voice testing