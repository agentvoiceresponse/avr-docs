---
title: AVR App – Administration Panel
description: Architecture, setup, runtime behavior, and operations guide for the AVR administration panel.
published: true
date: 2026-05-10T18:55:00.000Z
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

The AVR App repository contains two independent npm projects:

- **`backend/`**: NestJS 11 API (JWT auth, TypeORM + SQLite, Docker orchestration, ARI integration)
- **`frontend/`**: Next.js 16 + React 19 admin UI (Tailwind, shadcn/ui)

There is no root npm workspace. Run commands from each project directory.

---

## Requirements

- Docker & Docker Compose
- Git
- Node.js 20+

---

## Installation & Startup

Clone the application repository:

```bash
git clone https://github.com/agentvoiceresponse/avr-app.git
cd avr-app
```

Install dependencies:

```bash
cd backend && npm install
cd ../frontend && npm install
```

Run in development:

```bash
# terminal 1
cd backend
npm run start:dev

# terminal 2
cd frontend
npm run start:dev
```

Default local ports:

- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:3001`

---

## Optional Telephony Stack

This repo includes `docker-compose-asterisk.yml` to run local Asterisk-related services.

```bash
docker compose -f docker-compose-asterisk.yml up -d
```

---

## Login

Open:

`http://localhost:3000/login`

Default credentials:

- Username: **admin**
- Password: from `ADMIN_PASSWORD` (fallback default is `agentvoiceresponse`)

Admin user is seeded at backend startup using `ADMIN_USERNAME` and `ADMIN_PASSWORD`.

---

## Environment and Runtime Notes

- `FRONTEND_URL` controls CORS in backend (`main.ts`); wrong origin causes login/API failures.
- Frontend runtime env is read via `next-runtime-env` (`env('NEXT_PUBLIC_*')`), not `process.env`.
- Restart frontend dev server after editing `.env` values.
- `NEXT_PUBLIC_API_URL` must point to backend (default `http://localhost:3001`).
- Backend uses TypeORM with `synchronize: true`; schema changes can drop/reshape columns.
- Local SQLite path is `../data/data.db` (relative to backend).
- Backend needs Docker Engine running to spawn agent containers via Docker socket.

---

## Post-Setup Workflow

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

[https://demo.app.agentvoiceresponse.com/login](https://demo.app.agentvoiceresponse.com/login)

Viewer account:

- Username: discord
- Password: 05Tx14WbQN5oKQdx

---

## Repository Structure

- backend/
- frontend/
- docker-compose-asterisk.yml
- asterisk/
- data/

---

## Notes

- `GET /health` and `POST /auth/login` are public; other backend routes require JWT.
- Roles: `admin`, `manager`, `viewer`.
- Asterisk config files (`extensions.conf`, `pjsip.conf`, `manager.conf`) are rewritten by backend telephony operations.
