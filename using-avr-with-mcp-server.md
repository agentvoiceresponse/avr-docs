---
title: Using AVR with a Custom MCP Server
description: 
published: true
date: 2026-05-08T15:17:42.118Z
tags: 
editor: markdown
dateCreated: 2025-09-25T10:03:52.429Z
---

# Using AVR Wiki with a Custom MCP Server

ChatGPT supports integration with **custom MCP (Model Context Protocol) servers**, allowing you to extend ChatGPT’s capabilities with your own documentation or APIs.  

This feature makes it possible to query the **Agent Voice Response Wiki** directly from ChatGPT, without manually browsing the documentation.

---

## Why Use an MCP Server?

- 📖 Access AVR documentation directly inside ChatGPT  
- ⚡ Save time if you don’t want to search manually  
- 🧑‍💻 Ask in natural language (e.g. *“Show me an example docker-compose with AVR and OpenAI Realtime”*)  
- 🔗 Always stay aligned with the latest documentation  

---

## AVR Wiki MCP Server

We’ve exposed the MCP server for the AVR Wiki at:

👉 **https://wikimcp.agentvoiceresponse.com/mcp**

This server allows ChatGPT to fetch real-time answers directly from the AVR documentation.

---

## How to Configure MCP in ChatGPT

1. Open **ChatGPT** → go to **Settings → Beta features**.  
   - Enable **Custom GPTs** and **MCP Servers**.  

2. Go to **Settings → MCP Servers**.  

3. Add a new server:  
   - **Name**: `AVR Wiki MCP`  
   - **Endpoint**: `https://wikimcp.agentvoiceresponse.com/mcp`  

4. Save the configuration.  

5. Start a new chat — ChatGPT will now be able to query the AVR Wiki via the MCP server.

---

## Example Queries

Once configured, you can ask ChatGPT things like:

- *“How can I configure the docker-compose file to work with Agent Voice Response and OpenAI Realtime?”*  
- *“What are the environment variables for Gemini STS?”*  
- *“How do I integrate AVR with VitalPBX?”*  

ChatGPT will return the correct answer from the AVR documentation.

---

## Benefits

- 🕒 **Faster onboarding** — new users can ask ChatGPT instead of reading through the docs manually  
- 🧑‍💻 **Developer productivity** — answers in natural language, instantly  
- ✅ **Always accurate** — responses are sourced from the official AVR Wiki  

---

## Troubleshooting

- If ChatGPT doesn’t return results:  
  - Verify MCP server URL: `https://wikimcp.agentvoiceresponse.com/mcp`  
  - Make sure **MCP servers are enabled** in ChatGPT settings  
  - Restart the chat session after adding the server  