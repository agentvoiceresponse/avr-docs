---
title: Webhook Integration Guide
description: 
published: true
date: 2025-09-13T10:51:53.858Z
tags: 
editor: markdown
dateCreated: 2025-09-13T10:28:58.099Z
---

# Webhook Integration Guide

AVR Core provides comprehensive webhook support for real-time event tracking and integration with external systems. Webhooks allow you to receive instant notifications about call events, enabling you to build custom analytics, logging, and integration solutions.

## Overview

Webhooks are HTTP POST requests sent to your specified endpoint whenever specific events occur during a call session. This enables real-time monitoring, analytics, and integration with external systems like CRM platforms, analytics tools, or custom applications.

## Supported Webhook Events

### call_started
- **Trigger**: When a new call session begins
- **Payload**: Empty object `{}`
- **Use Case**: Initialize call tracking, start analytics, log call initiation

### call_ended
- **Trigger**: When a call session ends
- **Payload**: Empty object `{}`
- **Use Case**: Finalize call tracking, calculate call duration, update analytics

### interruption
- **Trigger**: When the AI response is interrupted by user speech
- **Payload**: Empty object `{}`
- **Use Case**: Track user engagement, measure interruption patterns, adjust AI behavior

### transcription
- **Trigger**: When speech is transcribed (STS mode only)
- **Payload**: Transcription object with role and text
- **Use Case**: Log conversations, analyze speech patterns, build conversation history

## Webhook Payload Format

All webhook requests follow a consistent JSON structure:

```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "call_started",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "payload": {
    // Event-specific data
  }
}
```

### Payload Fields

- **uuid**: Unique identifier for the call session
- **type**: Event type (call_started, call_ended, interruption, transcription)
- **timestamp**: ISO 8601 timestamp of when the event occurred
- **payload**: Event-specific data object

### Event-Specific Payloads

#### call_started
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "call_started",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "payload": {}
}
```

#### call_ended
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "call_ended",
  "timestamp": "2024-01-01T12:05:30.000Z",
  "payload": {}
}
```

#### interruption
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "interruption",
  "timestamp": "2024-01-01T12:02:15.000Z",
  "payload": {}
}
```

#### transcription
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "transcription",
  "timestamp": "2024-01-01T12:01:45.000Z",
  "payload": {
    "role": "user|agent",
    "text": "Hello, I need help with my account"
  }
}
```

## Configuration

### Environment Variables

Configure webhook settings using the following environment variables:

#### Required Configuration
- **WEBHOOK_URL**: The endpoint URL where webhook notifications will be sent
- **WEBHOOK_SECRET**: Secret key for webhook signature verification

#### Optional Configuration
- **WEBHOOK_TIMEOUT**: Request timeout in milliseconds (default: 3000ms)
- **WEBHOOK_RETRY**: Number of retry attempts for failed requests (default: 0)

### Example Configuration

```bash
# .env file
WEBHOOK_URL=https://your-domain.com/webhooks/avr
WEBHOOK_SECRET=your-secret-key-here
WEBHOOK_TIMEOUT=5000
WEBHOOK_RETRY=3
```

## Security

### Webhook Signature Verification

When `WEBHOOK_SECRET` is configured, AVR Core includes the secret in the `X-AVR-WEBHOOK-SECRET` header for verification:

```http
POST /webhooks/avr HTTP/1.1
Host: your-domain.com
Content-Type: application/json
X-AVR-WEBHOOK-SECRET: your-secret-key-here

{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "call_started",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "payload": {}
}
```

## Implementation Examples

- Github: https://github.com/agentvoiceresponse/avr-webhook

## Error Handling and Retries

### Automatic Retry Logic

AVR Core implements automatic retry logic for failed webhook requests:

1. **Initial Request**: Webhook is sent to the configured URL
2. **Timeout Handling**: If no response within `WEBHOOK_TIMEOUT`, request is considered failed
3. **Retry Attempts**: Up to `WEBHOOK_RETRY` additional attempts are made
4. **Exponential Backoff**: Retry attempts are spaced with increasing delays
5. **Final Failure**: After all retries are exhausted, the webhook is logged as failed

### Error Logging

Failed webhook requests are logged with detailed error information:

```
[WEBHOOK][CALL_STARTED] Retry failed
Webhook Error:
Message: connect ECONNREFUSED 127.0.0.1:3000
Code: ECONNREFUSED
URL: http://localhost:3000/webhooks/avr
Method: POST
Status: 
```


### Best Practices for Webhook Endpoints

1. **Fast Response**: Respond quickly (within timeout period)
2. **Idempotent Processing**: Handle duplicate webhook deliveries
3. **Error Handling**: Return appropriate HTTP status codes
4. **Logging**: Log all webhook events for debugging
5. **Security**: Verify webhook signatures when using secrets

### Common Issues and Solutions

#### Webhook Not Received
- **Check URL**: Verify `WEBHOOK_URL` is correct and accessible
- **Check Network**: Ensure network connectivity between AVR Core and webhook endpoint
- **Check Logs**: Review AVR Core logs for webhook delivery errors

#### Timeout Errors
- **Increase Timeout**: Set higher `WEBHOOK_TIMEOUT` value
- **Optimize Endpoint**: Improve webhook endpoint response time
- **Check Load**: Ensure webhook endpoint can handle request volume

#### Authentication Failures
- **Verify Secret**: Check `WEBHOOK_SECRET` configuration
- **Check Headers**: Ensure webhook endpoint reads `X-AVR-WEBHOOK-SECRET` header
- **Test Verification**: Implement and test signature verification logic

## Use Cases

### Call Analytics
Track call metrics, duration, and patterns

### CRM Integration
Update customer records with call information

### Quality Assurance
Monitor AI performance and user satisfaction