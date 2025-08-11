---
title: External Asterisk
description: Using AVR with External Asterisk PBX
published: true
date: 2025-08-11T13:17:58.504Z
tags: 
editor: markdown
dateCreated: 2025-08-08T16:44:58.314Z
---

# Using AVR with External Asterisk PBX

This guide explains how to configure **Asterisk PBX** so that calls are sent to **AVR Core** via the AudioSocket channel driver.

## 1. Prerequisites

Before proceeding, ensure you have:

- A working **Asterisk PBX** installation.
- The **AudioSocket channel driver** installed and enabled in Asterisk.
- The IP address and port of your **AVR Core** instance (e.g., `192.168.1.100:6000`).
- The `uuidgen` command available on your system (usually pre-installed on Linux).

## 2. Editing `extensions.conf`

Asterisk dial plans are configured in the `extensions.conf` file.  
By default, this file is located in:

```
/etc/asterisk/extensions.conf
```

## 3. Basic Configuration

**Option 1 – Direct AudioSocket Connection:**
```asterisk
same => n,AudioSocket(${UUID},IP_AVR_CORE:PORT_AVR_CORE)`
```

**Option 2 – Recommended for scalability (Dial syntax):**

```asterisk
same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})
```

Example extensions.conf:

```env
[avr-context]
exten => 5001,1,Answer()
 same => n,Ringing()
 same => n,Wait(1)
 same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
 same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})
 same => n,Hangup()
```
 

