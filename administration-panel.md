---
title: AVR App – Administration Panel
description: 
published: true
date: 2026-02-08T18:42:56.143Z
tags: asr, tts, avr, llm, sts, app, cloud, docker, admin
editor: markdown
dateCreated: 2025-09-30T14:23:03.468Z
---

# AVR App – Administration Panel

The **AVR App** is the official administration panel for **Agent Voice Response**.
It allows you to design, configure, deploy, and manage AI voice agents connected to Asterisk using a modern web interface.

## What You Can Do with the AVR App

- Design and manage AI voice agents
- Connect agents to ASR, LLM, TTS, and Speech-to-Speech providers
- Automatically manage Docker containers
- Configure phone numbers and call routing
- Test agents via SIP softphones or WebRTC
- Use tools / function calls (transfer, hangup, custom logic)
- Operate in a multi-tenant, role-based environment

## Architecture Overview

The AVR App stack includes:

- **Traefik** – Reverse proxy and router
- **avr-app-backend** – NestJS API (JWT, SQLite, Docker management, ARI)
- **avr-app-frontend** – Next.js 14 UI (Tailwind, shadcn/ui)
- **Asterisk PBX** – SIP, WebRTC, AudioSocket

All services are orchestrated via Docker Compose.

---

## Requirements

- Docker & Docker Compose
- Git

---

## Installation & Startup

Clone the repository:

```bash
git clone https://github.com/agentvoiceresponse/avr-infra.git
cd avr-infra
```

Start the stack:

```bash
docker compose -f docker-compose-app.yml up -d
```

---

## Access via Traefik

| Service | URL |
|------|-----|
| AVR App | http://avr.localhost |
| Backend API | http://api.localhost |
| Traefik Dashboard | http://localhost:8080 |

---

## Login

Open:

http://avr.localhost

Default credentials:

- Username: **admin**
- Password: **admin**

---

## Post-Installation Workflow

1. Create a Provider
2. Create an Agent
3. Verify Containers
4. Create a Number (e.g. 6001)
5. Associate the Agent

---

## Testing

### SIP Softphone

- Username: 1000
- Password: 1000
- Call: 6001

### WebRTC Phone

- Password: 2000
- Accept certificate at: https://localhost:8089/ws

---

## Demo

https://demo.app.agentvoiceresponse.com/login

Viewer account:

- Username: discord
- Password: 05Tx14WbQN5oKQdx

---

## Repository Structure

- backend/
- frontend/
- docker-compose-app.yml
- asterisk/
- data/

---

## Notes

The AVR App provides a production-like environment locally and is the foundation for scalable conversational AI voice systems.
