---
title: Using AVR with FreePBX
description: 
published: true
date: 2025-08-20T22:49:34.055Z
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

```bash
asterisk -rx "module show like audiosocket"
```

Perfetto 👍 Ti ho preparato una guida completa in Markdown seguendo lo stesso stile delle pagine precedenti della documentazione AVR.

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

⸻

2. Get the AVR Infrastructure

git clone https://github.com/agentvoiceresponse/avr-infra.git
cd avr-infra


⸻

3. Configure AVR
	1.	Edit the .env file and set the required values (API keys, ports, etc.).
	2.	Open the relevant docker-compose-*.yml file.
	3.	Comment out the entire avr-asterisk service block (since you are using FreePBX’s Asterisk).

⸻

4. Launch AVR

For example, with Ultravox:

docker-compose -f docker-compose-ultravox.yml up -d


⸻

5. Confirm AVR is Listening on Port 5001

ss -lntp | grep :5001


⸻

6. FreePBX Configuration (GUI)

Go to the FreePBX administration interface and configure as follows:

6.1 Custom Dialplan
	•	Navigate to:
Admin → Config Edit → Asterisk Custom Configuration Files → extensions_custom.conf
	•	Append:

[ultravox-sts]
exten => 7000,1,NoOp(AVR <-> Ultravox STS)
 same => n,Answer()
 same => n,Set(AS_UUID=${SHELL(/usr/bin/uuidgen | /usr/bin/tr -d '\r\n')})
 same => n,AudioSocket(${AS_UUID},127.0.0.1:5001)
 same => n,Hangup()


⸻

6.2 Custom Destination
	•	Go to: Admin → Custom Destinations
	•	Add:
	•	Target: ultravox-sts,7000,1
	•	Description: Ultravox

⸻

6.3 Inbound Route
	•	Go to: Connectivity → Inbound Routes
	•	Set Destination to:
Custom Destinations → Ultravox
	•	Click Apply Config.

⸻

7. Testing the Setup
	1.	Call the DID that is routed to the Ultravox Custom Destination.
	2.	Verify that audio flows correctly to/from AVR via AudioSocket.

⸻

Troubleshooting
	•	Ensure the AudioSocket port in the FreePBX dialplan matches the port exposed by the AVR container (default: 5001).
	•	Check container logs with:

docker logs avr-core


	•	Use:

asterisk -rvvv

to view Asterisk console output in real time.

⸻

✅ With this configuration, FreePBX routes calls to AVR Core over AudioSocket, enabling full integration with your ASR/LLM/TTS pipeline.

Vuoi che aggiungo anche uno **schema grafico in Mermaid** (tipo: `FreePBX → AVR Core → ASR/TTS/LLM`) per mantenere lo stile coerente con le altre sezioni?