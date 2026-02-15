---
title: AVR Core HTTP Web Service
description: 
published: true
date: 2026-02-15T11:03:25.464Z
tags: 
editor: markdown
dateCreated: 2026-02-15T11:03:25.464Z
---

# AVR Core HTTP Web Service

AVR Core includes an **optional HTTP web service** that enables call initialization, metadata propagation, and **dynamic Speech-to-Speech (STS) routing**.

When enabled, the HTTP server runs **alongside** the AudioSocket TCP server and does not alter the classic AVR execution model.

## Overview

The HTTP Web Service allows `avr-core` to:

* initialize a call from the Asterisk dialplan
* receive and persist **call metadata**
* build the effective STS connection URL
* optionally delegate STS routing decisions to an external webhook

If the HTTP service is **disabled**, AVR Core continues to operate **exclusively as an AudioSocket server**, preserving full backward compatibility.

## Enabling the HTTP Server

The HTTP server is enabled by setting the `HTTP_PORT` environment variable.

### Environment Variables

| Variable    | Description                          | Required             |
| ----------- | ------------------------------------ | -------------------- |
| `HTTP_PORT` | Port used to expose the HTTP service | No                   |
| `PORT`      | AudioSocket TCP port                 | Yes (default: `5001`) |

### Example

```bash
HTTP_PORT=6001
PORT=5001
```

### Expected Logs

```text
[INFO]: AVR HTTP Server listening on port 6001
[INFO]: AVR AudioSocket Server listening on port 5001
```

## HTTP Endpoints

### `GET /health`

Basic health check endpoint.

**Response**

* `200 OK`

```json
{ "status": "ok" }
```

### `POST /call`

Call initialization endpoint, typically invoked from the **Asterisk dialplan** using the `CURL()` function.

This endpoint is responsible for:

* registering call metadata
* computing the effective STS URL
* emitting a webhook event (optional)
* enabling dynamic STS routing

## Request

### Body

```json
{
  "uuid": "935652e3-7c59-4edc-b17f-9646a9c2a265",
  "payload": {
    "from": "2000",
    "to": "5001",
    "uniqueid": "1770758888.32",
    "channel": "PJSIP/2000-00000013"
  }
}
```

### Fields

| Field     | Description                                              |
| --------- | -------------------------------------------------------- |
| `uuid`    | Unique identifier correlating HTTP, AudioSocket, and STS |
| `payload` | Free-form object containing call metadata                |

All key/value pairs inside `payload`:

* are stored as call metadata
* are forwarded to the `call_initiated` webhook
* are propagated to the STS endpoint as **query parameters**

## Server Behavior

Upon receiving `POST /call`, AVR Core performs the following steps:

### 1. Effective STS URL Construction

Starting from `STS_URL`, AVR Core appends all `payload` fields as query parameters.

Example:

```text
STS_URL = ws://localhost:6030
```

becomes:

```text
ws://localhost:6030/?from=2000&to=5001&uniqueid=1770758888.32&channel=...
```

### 2. Webhook Emission (`call_initiated`)

If `WEBHOOK_URL` is configured, AVR Core sends an HTTP webhook with:

* `type = call_initiated`
* the same `payload` received from `/call`

### 3. Dynamic STS Override (Optional)

If the webhook HTTP response contains a `sts_url` field:

* the configured `STS_URL` is **overridden**
* payload query parameters are still appended

This mechanism enables fully dynamic STS routing.

> **Note**
> `STS_URL` must still be configured, as it acts as a base and fallback value.

## Response

```json
{
  "sts_url": "ws://localhost:6030/?from=2000&to=5001&uniqueid=1770758888.32&channel=PJSIP%2F2000-00000013",
  "override": false
}
```

### Fields

| Field      | Description                                               |
| ---------- | --------------------------------------------------------- |
| `sts_url`  | Final STS WebSocket URL used for the call                 |
| `override` | `true` if provided by webhook, `false` if locally derived |


## Asterisk Dialplan Example

The following example:

1. generates a UUID
2. calls `POST /call` with call metadata
3. opens the AudioSocket connection using the same UUID

```asterisk
exten => 5001,1,Answer()
same => n,Ringing()
same => n,Wait(1)
same => n,Set(UUID=${SHELL(uuidgen | tr -d '\n')})
same => n,Set(JSON_BODY={"uuid":"${UUID}","payload":{"from":"${CALLERID(num)}","to":"${EXTEN}","uniqueid":"${UNIQUEID}","channel":"${CHANNEL}"}})
same => n,Set(CURLOPT(httpheader)=Content-Type: application/json)
same => n,Set(JSON_RESPONSE=${CURL(http://avr-core:6001/call,${JSON_BODY})})
same => n,NoOp(JSON_BODY: ${JSON_BODY})
same => n,NoOp(JSON_RESPONSE: ${JSON_RESPONSE})
same => n,Set(DENOISE(rx)=on)
same => n,Dial(AudioSocket/avr-core:5001/${UUID})
same => n,Hangup()
```

### Adjust as needed

* `http://avr-core:6001` → `http://<avr-core-host>:<HTTP_PORT>`
* `AudioSocket/avr-core:5001` → `AudioSocket/<avr-core-host>:<PORT>`

## Metadata Propagation to STS

When `STS_URL` uses `ws://` or `wss://`, AVR Core connects via WebSocket and passes metadata as query parameters.

### Example URL

```text
ws://localhost:6030/?from=2000&to=5001&uniqueid=1770758888.32&channel=PJSIP%2F2000-00000013
```

### Example Log

```text
[INFO]: Connecting to STS WebSocket: ws://localhost:6030/?from=2000&to=5001&uniqueid=1770758888.32&channel=PJSIP%2F2000-00000013
```

### Reading Parameters in STS (Node.js)

```js
wss.on("connection", (clientWs, request) => {
  const url = new URL(request.url, `http://${request.headers.host}`);
  const from = url.searchParams.get("from");
  const to = url.searchParams.get("to");
  const uniqueid = url.searchParams.get("uniqueid");
  const channel = url.searchParams.get("channel");
  console.log({ from, to, uniqueid, channel });
});
```

## Webhook Event: `call_initiated`

### Payload

```json
{
  "uuid": "935652e3-7c59-4edc-b17f-9646a9c2a265",
  "type": "call_initiated",
  "timestamp": "2026-01-01T12:00:00.000Z",
  "payload": {
    "from": "2000",
    "to": "5001",
    "uniqueid": "1770758888.32",
    "channel": "PJSIP/2000-00000013"
  }
}
```

### Dynamic Routing via Webhook Response

If the webhook responds with:

```json
{
  "sts_url": "ws://localhost:6030?uuid=935652e3-7c59-4edc-b17f-9646a9c2a265"
}
```

AVR Core will use that URL for the current call.

### Minimal Express Example

```js
const express = require("express");
const app = express();
app.use(express.json());

app.post("/events", (req, res) => {
  if (req.body.type === "call_initiated") {
    return res.json({
      sts_url: `ws://localhost:6030?uuid=${req.body.uuid}`,
    });
  }
  res.sendStatus(200);
});

app.listen(process.env.PORT || 9000);
```

### Expected Logs

```text
[INFO]: Webhook response received: {"sts_url":"ws://localhost:6030?uuid=..."}
[INFO]: Overriding STS URL: ws://localhost:6030?uuid=...
```

## Why This Matters

Dynamic STS routing allows you to:

* use a **single AVR Core instance** with multiple STS backends
* route calls based on:

  * caller (`from`)
  * service or language (`to`)
  * any custom business logic
* correlate AVR events with Asterisk CDRs via:

  * `uniqueid`
  * `channel`

## Security Considerations

The HTTP endpoints do **not** provide built-in authentication.

It is strongly recommended to:

* expose them only on trusted networks
* use internal Docker networks
* protect access via a reverse proxy with authentication
* apply firewall or network ACLs