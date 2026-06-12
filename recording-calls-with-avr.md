---
title: Recording Calls with AVR
description: How to record AVR calls using Asterisk MixMonitor — the recommended production approach when AVR Core and STS connectors do not provide native recording.
published: true
date: 2026-06-12T19:30:00.000Z
tags: avr, asterisk, recording, mixmonitor, compliance
editor: markdown
dateCreated: 2026-06-12T19:30:00.000Z
---

# Recording Calls with AVR

AVR does **not** currently provide a native call recording feature inside `avr-core` or STS connectors such as `avr-sts-openai`. There is no hidden environment variable, debug option, or undocumented flag for recording calls at the STS connector level.

Because AVR is built on top of **Asterisk**, the most reliable and production-ready way to record calls is to use the standard Asterisk **`MixMonitor`** application in your dialplan **before** the call is handed to AVR Core through `Dial(AudioSocket/...)`.

## Recommended approach

Record at the **Asterisk layer**. Asterisk owns the call and media session before audio is passed to AVR through AudioSocket. This keeps AVR focused on real-time audio processing and agent orchestration, while PBX-level concerns such as recording, retention, storage, and compliance policies stay with Asterisk and your surrounding infrastructure.

> **Note:** The HTTP service in `avr-core` 1.12+ is not intended to replace Asterisk-level recording. For compliance-grade call recording, Asterisk remains the right layer.

## Basic dialplan example

If your current dialplan contains something like:

```asterisk
same => n,Dial(AudioSocket/avr-core:5001/${UUID})
```

Add `MixMonitor` **before** the `Dial` application:

```asterisk
same => n,MixMonitor(/var/spool/asterisk/monitor/${UUID}.wav)
same => n,Dial(AudioSocket/avr-core:5001/${UUID})
```

## Complete dialplan example

```asterisk
exten => _X.,1,NoOp(Starting AVR call with recording)
 same => n,Set(UUID=${UNIQUEID})
 same => n,MixMonitor(/var/spool/asterisk/monitor/${UUID}.wav)
 same => n,Dial(AudioSocket/avr-core:5001/${UUID})
 same => n,Hangup()
```

This creates one `.wav` recording per call under:

```bash
/var/spool/asterisk/monitor/
```

Use a unique filename per call (`${UUID}`, `${UNIQUEID}`, or another stable identifier) so recordings do not overwrite each other.

## Docker and `avr-asterisk`

If you use **`avr-asterisk`** (or the Asterisk service from `avr-infra`), Asterisk runs inside Docker. Mount the recording directory as a volume so files are available on the host:

```yaml
services:
  avr-asterisk:
    volumes:
      - ./recordings:/var/spool/asterisk/monitor
```

With this setup, recordings created by Asterisk appear on the host in `./recordings`.

The `avr-infra` **AVR App** stack (`docker-compose-app.yml`) already mounts `./asterisk/recordings` into both Asterisk and the app backend:

```yaml
avr-app-backend:
  environment:
    - ASTERISK_RECORDINGS_PATH=/app/recordings
  volumes:
    - ./asterisk/recordings:/app/recordings

avr-asterisk:
  volumes:
    - ./asterisk/recordings:/var/spool/asterisk/monitor
```

Point `MixMonitor` at `/var/spool/asterisk/monitor/${UUID}.wav` inside Asterisk so filenames align with what the backend expects (`{callUuid}.wav`).

## Browsing recordings in AVR App

When you run the **AVR App** administration panel, the **Recordings** page lists `.wav` files from the Asterisk monitor directory. The backend reads files from `ASTERISK_MONITOR_PATH` (default `../recordings` locally, `/app/recordings` in the App compose stack).

To enable recordings in the UI:

1. Add `MixMonitor` to your Asterisk dialplan (see examples above).
2. Ensure the monitor directory is mounted into both Asterisk and `avr-app-backend`.
3. Name files `{callUuid}.wav` where `callUuid` matches the UUID passed to AVR Core in the `Dial(AudioSocket/.../${UUID})` path.

Open **Recordings** in the sidebar to browse, play, and download stored calls.

## External Asterisk or FreePBX / VitalPBX

The same `MixMonitor` pattern applies when AVR connects to an **external Asterisk** PBX. Edit `extensions.conf` (or the equivalent dialplan in your platform) on the PBX that routes calls to AVR, add `MixMonitor` before the AudioSocket `Dial`, and configure retention or export on that host.

See also:

- [Using AVR with External Asterisk PBX](/external-asterisk)
- [Integrating VitalPBX with AVR](/integrating-vitalpbx-with-avr)
- [Using AVR with FreePBX](/using-avr-with-freepbx)

## What is not supported today

| Area | Status |
|------|--------|
| Native recording in `avr-core` | Not available |
| Recording via STS connectors (e.g. OpenAI Realtime) | Not available |
| Stable public API for raw audio frame capture during a call | Not exposed for recording use cases |

Native recording inside AVR Core may be considered in the future, but **do not depend on it for production today**. Plan around Asterisk `MixMonitor`.

## Summary

For a simple, reliable recording setup:

```asterisk
same => n,MixMonitor(/var/spool/asterisk/monitor/${UUID}.wav)
same => n,Dial(AudioSocket/avr-core:5001/${UUID})
```

With Docker:

```yaml
volumes:
  - ./recordings:/var/spool/asterisk/monitor
```

This gives you one recording file per call, host-accessible storage, and optional browsing through AVR App when the stack and filenames are aligned.
