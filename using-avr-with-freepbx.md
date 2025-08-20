---
title: Using AVR with FreePBX
description: 
published: true
date: 2025-08-20T23:05:58.033Z
tags: 
editor: markdown
dateCreated: 2025-08-20T22:49:34.055Z
---

# Using AVR with FreePBX (AudioSocket)

**Tested on:** FreePBX 17  
**Also likely compatible with:** FreePBX 15 / 16  

The goal of this guide is to connect **FreePBX/Asterisk** to **AVR (Agent Voice Response)** over **AudioSocket**.

---

## Prerequisites

- A working **FreePBX installation** (ISO, install script, or manual).
  - FreePBX ISO/install script usually enables `res_audiosocket` by default.
  - For manual installs, ensure `res_audiosocket.so` is loaded.
- **Docker** and **Docker Compose** (v2 plugin or `docker-compose`).
- **Git** installed.

---

## Verify AudioSocket Module (optional)

You can check if the module is loaded by running:

```console
asterisk -rx "module show like audiosocket"
```

## Install Docker & Docker Compose

Example for Debian/Ubuntu:

```console
sudo apt-get update && apt-get install -y docker.io docker-compose-plugin
sudo systemctl enable --now docker
```

## Get the AVR Infrastructure

```console
git clone https://github.com/agentvoiceresponse/avr-infra.git
cd avr-infra
```

## Configure AVR
1.	Edit the .env file and set the required values (API keys, ports, etc.).
2.	Open the relevant docker-compose-*.yml file.
3.	Comment out the entire avr-asterisk service block (since you are using FreePBX’s Asterisk).

## Launch AVR

For example, with Ultravox:

```console
docker-compose -f docker-compose-ultravox.yml up -d
```

## Confirm AVR is Listening on Port 5001

```console
ss -lntp | grep :5001
```

## FreePBX Configuration (GUI)

Go to the FreePBX administration interface and configure as follows:

### Custom Dialplan
- Navigate to:
```
Admin → Config Edit → Asterisk Custom Configuration Files → 
extensions_custom.conf

```

- Append:

```env
[ultravox-sts]
exten => 7000,1,NoOp(AVR <-> Ultravox STS)
 same => n,Answer()
 same => n,Set(AS_UUID=${SHELL(/usr/bin/uuidgen | /usr/bin/tr -d '\r\n')})
 same => n,AudioSocket(${AS_UUID},127.0.0.1:5001)
 same => n,Hangup()
```

### Custom Destination
- Go to: Admin → Custom Destinations
- Add:
	- Target: ultravox-sts,7000,1
	- Description: Ultravox

### Inbound Route
- Go to: Connectivity → Inbound Routes
- Set Destination to:
```
Custom Destinations → Ultravox
```
- Click **Apply** Config

## Testing the Setup
1.	Call the DID that is routed to the Ultravox Custom Destination.
2.	Verify that audio flows correctly to/from AVR via AudioSocket.

## Troubleshooting
- Ensure the AudioSocket port in the FreePBX dialplan matches the port exposed by the AVR container (default: 5001).
- Check container logs with:
```console
docker logs avr-core
```

- Use:
```console
asterisk -rvvv
```
to view Asterisk console output in real time.


---


With this configuration, FreePBX routes calls to AVR Core over AudioSocket, enabling full integration with your ASR/LLM/TTS pipeline.