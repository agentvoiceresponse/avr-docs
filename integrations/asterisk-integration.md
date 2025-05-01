---
title: asterisk-integration
description: 
published: true
date: 2025-05-01T19:20:00.664Z
tags: 
editor: markdown
dateCreated: 2025-05-01T19:09:42.360Z
---

# Asterisk Integration Guide

This guide explains how to integrate AVR with Asterisk, including basic setup, FreePBX integration, and best practices for audio handling.

## Prerequisites

- Asterisk 20+
- AVR Core and provider integrations
- Basic understanding of Asterisk dialplan
- Network access between Asterisk and AVR services

## Basic Asterisk Setup

### 1. Install Asterisk with AudioSocket Module

```bash
cd /usr/src
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
tar xvf asterisk-20-current.tar.gz
cd asterisk-20.*/contrib/ast_tools
make
```

### 2. Configure Asterisk

Add the following to your `extensions.conf`:

```ini
[from-internal]
exten => _6XXX,1,Answer()
exten => _6XXX,1,Ringing()
exten => _6XXX,n,AudioSocket(127.0.0.1,5001)
exten => _6XXX,n,Hangup()
```

### 3. Configure PJSIP

Add the following to your `pjsip.conf`:

```ini
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060

[1000]
type=endpoint
context=from-internal
disallow=all
allow=ulaw
allow=alaw
auth=1000
aors=1000

[1000]
type=auth
auth_type=userpass
password=your_password
username=1000

[1000]
type=aors
max_contacts=1
```

## FreePBX Integration

### 1. Install FreePBX with AudioSocket Module

```bash
```

### 2. Create Custom Destination

1. Go to FreePBX Admin
2. Navigate to Admin → Custom Destinations
3. Click "Add Custom Destination"
4. Enter the following:
   - Description: AVR Integration
   - Custom Destination: `custom-avr,s,1`
   - Custom Context: `from-internal-custom`

### 3. Configure Custom Context

Add the following to `extensions_custom.conf`:

```ini
[from-internal-custom]
exten => s,1,Answer()
exten => s,n,Ringing()
exten => s,n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
exten => s,n,AudioSocket(${UUID},AVR_HOST_IP_OR_DOMAIN:5001)
exten => s,n,Hangup()
```

### 4. Create Inbound Route

1. Go to FreePBX Admin
2. Navigate to Connectivity → Inbound Routes
3. Click "Add Inbound Route"
4. Configure:
   - Description: AVR Inbound
   - DID Number: Your DID
   - Destination: Custom Destinations → AVR Integration

## Audio Handling

### 1. Audio Formats

AVR supports the following audio formats:
- G.711 (ulaw/alaw)
- Linear PCM (16-bit)
- Sample rate: 8kHz or 16kHz

### 2. AudioSocket Configuration

```ini
[general]
format=ulaw
samplerate=8000
```

### 3. Audio Processing

```ini
[from-internal]
exten => _6XXX,1,Answer()
exten => _6XXX,1,Ringing()
exten => _6XXX,1,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
exten => _6XXX,n,AudioSocket(${UUID},AVR_HOST_IP_OR_DOMAIN:5001)
exten => _6XXX,n,Hangup()
```

## Monitoring and Debugging

### 1. Asterisk CLI Commands

```bash
# Check AudioSocket status
asterisk -rx "core show applications"

# Monitor calls
asterisk -rx "core show channels"

# Check audio quality
asterisk -rx "core show settings"
```

### 2. Logging

Add to `logger.conf`:

```ini
[general]
dateformat=%F %T
msdateformat=%F %T.%L
appendhostname=yes
queue_log=yes
rotatestrategy=rotate
exec_after_rotate=gzip -9 ${filename}.2

[logfiles]
console => notice,warning,error,debug
messages => notice,warning,error
full => notice,warning,error,debug,verbose
```

## Best Practices

1. **Always Use Answer()**
   - Always call `Answer()` before `AudioSocket()`
   - This ensures proper audio channel setup

2. **Timeout Handling**
   - Set appropriate timeouts
   - Handle timeout scenarios gracefully

3. **Error Recovery**
   - Implement retry mechanisms
   - Provide fallback options

4. **Audio Quality**
   - Use appropriate codecs
   - Monitor audio quality
   - Handle network issues

5. **Security**
   - Restrict access to AudioSocket
   - Use secure connections
   - Monitor for abuse

## Example Configurations

### 1. Basic AVR Integration

```ini
[from-internal]
exten => _6XXX,1,Answer()
exten => _6XXX,n,AudioSocket(127.0.0.1,6000)
exten => _6XXX,n,Hangup()
```

### 2. Advanced AVR Integration

```ini
[from-internal]
exten => _6XXX,1,Answer()
exten => _6XXX,n,Set(TIMEOUT(absolute)=60)
exten => _6XXX,n,Set(MAXRETRIES=3)
exten => _6XXX,n,Set(RETRYCOUNT=0)

exten => _6XXX,n(retry),AudioSocket(127.0.0.1,6000)
exten => _6XXX,n,GotoIf($["${AUDIOSOCKET_STATUS}" = "SUCCESS"]?hangup)
exten => _6XXX,n,Set(RETRYCOUNT=$[${RETRYCOUNT} + 1])
exten => _6XXX,n,GotoIf($[${RETRYCOUNT} < ${MAXRETRIES}]?retry)
exten => _6XXX,n,Playback(pls-try-again)
exten => _6XXX,n,Hangup()
```

### 3. FreePBX Custom Destination

```ini
[from-internal-custom]
exten => s,1,Answer()
exten => s,n,Set(TIMEOUT(absolute)=60)
exten => s,n,AudioSocket(127.0.0.1,6000)
exten => s,n,GotoIf($["${AUDIOSOCKET_STATUS}" = "SUCCESS"]?hangup)
exten => s,n,Playback(pls-try-again)
exten => s,n,Hangup()
```

## Troubleshooting

### Common Issues

1. **No Audio**
   - Check if `Answer()` is called before `AudioSocket()`
   - Verify network connectivity
   - Check audio format compatibility

2. **Poor Audio Quality**
   - Verify codec settings
   - Check network latency
   - Monitor system resources

3. **Connection Issues**
   - Verify IP and port settings
   - Check firewall rules
   - Test network connectivity

### Debug Commands

```bash
# Check AudioSocket status
asterisk -rx "core show applications"

# Monitor calls
asterisk -rx "core show channels"

# Check audio quality
asterisk -rx "core show settings"

# View logs
tail -f /var/log/asterisk/full
```

## Next Steps

1. **Testing**
   - Test with different codecs
   - Verify error handling
   - Check performance

2. **Monitoring**
   - Set up monitoring
   - Configure alerts
   - Track metrics

3. **Optimization**
   - Fine-tune settings
   - Optimize performance
   - Improve reliability 