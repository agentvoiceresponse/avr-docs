---
title: AVR App – Administration Panel
description: 
published: true
date: 2025-09-30T14:25:22.630Z
tags: asr, tts, avr, llm, sts, app, cloud, docker, admin
editor: markdown
dateCreated: 2025-09-30T14:23:03.468Z
---

# AVR App – Administration Panel

The **AVR App** is the official **administration panel** for Agent Voice Response.  
It allows you to **design, train, and orchestrate AI voice agents** in a single dashboard and connect them to your preferred ASR, LLM, TTS, or real-time speech providers with just a few clicks.  

## Overview

- **Visual Agent Builder** → Build agents faster using reusable templates and provider presets  
- **One-Click Integrations** → Supports OpenAI, Deepgram, ElevenLabs, Ultravox, Anthropic, Ollama, and more  
- **Guided Workflows** → Eliminate complex setup, so teams can go from idea to production in record time  
- **Collaboration Ready** → Secure multi-tenant environment with role-based access controls  

👉 Try the live demo here: [AVR App Demo](https://demo.agentvoiceresponse.com/login)  

Login with account using role **viewer**,

- Username: discord
- Password: 05Tx14WbQN5oKQdx

## Architecture

The repository is organized into three main components:

- **`backend/`** → NestJS API with TypeORM + SQLite, JWT authentication, Docker management  

- **`frontend/`** → Next.js 14 web UI with Tailwind CSS, shadcn/ui, light/dark mode  

- **`docker-compose.yml`** → Orchestrates backend and frontend services  