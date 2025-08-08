---
title: External Asterisk
description: Using AVR with External Asterisk PBX
published: true
date: 2025-08-08T16:54:56.405Z
tags: 
editor: markdown
dateCreated: 2025-08-08T16:44:58.314Z
---

# Using AVR with External Asterisk PBX

Configure Asterisk extensions to connect with avr-core using:

`same => n,AudioSocket(${UUID},IP_AVR_CORE:PORT_AVR_CORE)`

**or (recommended for scalability):**

`same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})`

Example extensions.conf:

`[avr-context]`
`exten => 5001,1,Answer()`
` same => n,Ringing()`
` same => n,Wait(1)`
` same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})`
` same => n,Dial(AudioSocket/IP_AVR_CORE:PORT_AVR_CORE/${UUID})`
` same => n,Hangup()`

