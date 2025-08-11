---
title: External Asterisk
description: Using AVR with External Asterisk PBX
published: true
date: 2025-08-11T13:21:33.490Z
tags: 
editor: markdown
dateCreated: 2025-08-08T16:44:58.314Z
---

# Using AVR with External Asterisk PBX

This guide explains how to configure **Asterisk PBX** so that calls are sent to **AVR Core** via the AudioSocket channel driver.

## Prerequisites

Before proceeding, ensure you have:

- A working **Asterisk PBX** installation.
- The **AudioSocket channel driver** installed and enabled in Asterisk.
- The IP address and port of your **AVR Core** instance (e.g., `192.168.1.100:6000`).
- The `uuidgen` command available on your system (usually pre-installed on Linux).

## Editing `extensions.conf`

Asterisk dial plans are configured in the `extensions.conf` file.  
By default, this file is located in:

```
/etc/asterisk/extensions.conf
```

## Basic Configuration

**Option 1 – Direct AudioSocket Connection:**
```asterisk
same => n,AudioSocket(${UUID},IP_AVR_CORE:PORT_AVR_CORE)`
```

**Option 2 – Recommended for scalability (Dial syntax):**

```asterisk
same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})
```

The Dial method is recommended because it integrates better with Asterisk’s native call handling, enabling more flexible call routing and scalability.

## Example: extensions.conf

Below is a minimal working example that answers a call, generates a UUID, and connects it to AVR Core.

```env
[avr-context]
exten => 5001,1,Answer()
 same => n,Ringing()
 same => n,Wait(1)
 same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
 same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})
 same => n,Hangup()
```
 
Explanation:
- exten => 5001,1,Answer() → Answers the incoming call.
- Wait(1) → Adds a short delay to ensure the call is stable before connecting.
- Set(UUID=...) → Creates a unique identifier for the session, used by AVR Core to track the call.
- Dial(...) → Connects to AVR Core using AudioSocket.
- Hangup() → Ends the call when AVR Core terminates the session.

## Reloading Asterisk

After editing extensions.conf, reload the Asterisk dialplan:

```bash
asterisk -rx "dialplan reload"
```

## Testing

1. Dial 5001 (or the extension you configured) from any endpoint connected to your Asterisk PBX.
2. The call should route to AVR Core and start interacting via your configured ASR/LLM/TTS or STS modules.
3. Check Asterisk logs for troubleshooting:

```bash
asterisk -rvvvvv
```
  
> **Tip**: For production deployments, you may want to place AVR Core on a dedicated network interface or behind a load balancer for better scalability.
  
