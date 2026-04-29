# REQ-05: MCP Support (Model Context Protocol)

**Area:** Capabilities  
**Priority:** P1 — Should Have  
**Related:** [03_tool_system.md](03_tool_system.md) | [04_skills.md](04_skills.md) | [09_llm_providers.md](09_llm_providers.md)

---

## Overview

MCP (Model Context Protocol) is an open standard by Anthropic for connecting AI agents
to external tools, data sources, and services through a unified protocol.

The SDK MUST support MCP as a first-class integration — allowing agents to consume
any MCP-compatible server as a source of tools, resources, and prompts.

---

## MCP Architecture

```
Agent SDK (MCP Client)
       │
       │  JSON-RPC over stdio / HTTP / SSE
       ▼
  MCP Server
  ├── Tools     (callable functions)
  ├── Resources (data sources: files, DBs, APIs)
  └── Prompts   (reusable prompt templates)
```

---

## Requirements

### R5.1 — MCP Client Integration
- The SDK MUST include a built-in MCP client
- The client MUST support all MCP transport types:
  - `stdio` (local process)
  - `HTTP + SSE` (remote server)
  - `WebSocket` (streaming remote)

### R5.2 — Tool Auto-Discovery
- On connecting to an MCP server, the SDK MUST:
  1. Call `tools/list` to discover all available tools
  2. Automatically register discovered tools into the agent's tool registry
  3. Map MCP tool schemas to the SDK's internal tool schema format

### R5.3 — Resource Access
- The SDK MUST support MCP `resources/read` to fetch context from data sources
- Resources MUST be injectable into agent memory or context window

### R5.4 — Prompt Templates
- The SDK MUST support MCP `prompts/get` to load prompt templates from servers
- These templates SHOULD be usable as skill prompt addons (see [04_skills.md](04_skills.md))

### R5.5 — Multiple MCP Servers
- An agent MUST be able to connect to multiple MCP servers simultaneously
- Tools from all connected servers MUST be namespaced to avoid collisions
  - e.g., `github::create_issue`, `notion::create_page`

### R5.6 — MCP Server as Skill
- A connected MCP server SHOULD be wrappable as a Skill for easy reuse across agents

### R5.7 — Authentication
- The MCP client MUST support auth headers (API keys, OAuth tokens) for remote servers
- Credentials MUST be configurable via environment variables or a secrets manager

### R5.8 — Lifecycle Management
- MCP connections MUST be established during agent initialization
- Connections MUST be cleanly closed when the agent run ends
- Connection failures MUST not crash the agent — they MUST surface as a config error

---

## MCP Config Example (Conceptual)

```python
Agent(
  name="my-agent",
  mcp_servers=[
    MCPServer(name="github", transport="stdio", command="npx @modelcontextprotocol/server-github"),
    MCPServer(name="notion",  transport="http",  url="https://mcp.notion.com", api_key="..."),
  ]
)
```

---

## Ecosystem Support

MCP is supported today by: Claude, ChatGPT, VS Code Copilot, Cursor, and 100s of servers in the community registry.

---

*← [Back to Index](00_INDEX.md)*
