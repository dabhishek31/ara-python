# REQ-03: Tool System

**Area:** Capabilities  
**Priority:** P0 — Must Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [05_mcp_support.md](05_mcp_support.md) | [11_human_in_the_loop.md](11_human_in_the_loop.md) | [14_error_handling.md](14_error_handling.md)

---

## Overview

Tools are callable functions that extend the agent's ability to interact with the world.
A tool wraps a real action (API call, DB query, file read, web search, code execution, etc.)
and exposes it to the LLM in a structured, schema-validated way.

---

## Tool Anatomy

```
Tool
 ├── name         (string, snake_case)
 ├── description  (string — what the tool does, in plain language for the LLM)
 ├── input_schema (JSON Schema / Pydantic model)
 ├── output_schema (optional — for structured return types)
 ├── handler      (the actual function to call)
 ├── needs_approval (bool — triggers HITL if true)
 └── metadata     (tags, timeout, retry policy)
```

---

## Requirements

### R3.1 — Tool Registration
- Tools MUST be registerable via a decorator or explicit registration API
- Example: `@tool(name="search_web", description="...")`
- Tools MUST auto-generate JSON Schema from type hints / Pydantic models

### R3.2 — Schema Validation
- Input arguments from the LLM MUST be validated against the tool's schema before execution
- Invalid inputs MUST return a structured error back to the LLM (not crash the agent)

### R3.3 — Sync and Async Tools
- Tools MUST support both sync (`def`) and async (`async def`) handlers
- The executor MUST handle both transparently

### R3.4 — Tool Output
- Tool output MUST be serializable (string, dict, or Pydantic model)
- Output MUST be injected into the agent context as an **Observation**

### R3.5 — Tool Timeout
- Each tool MUST support a configurable `timeout` (default: 30s)
- On timeout, a `ToolTimeoutError` MUST be raised and returned as an observation

### R3.6 — Human-Approval Flag
- Tools MAY be marked `needs_approval=True`
- When such a tool is called, execution MUST pause for human approval (see [11_human_in_the_loop.md](11_human_in_the_loop.md))

### R3.7 — Built-in Tool Types
The SDK MUST ship with built-in tool adapters for common use cases:
- **Web Search** (e.g., Google Search, Tavily)
- **Code Execution** (sandboxed Python runner)
- **File Read/Write**
- **HTTP Request** (generic REST caller)
- **Memory Lookup** (query agent memory)
- **Agent-as-Tool** (call a sub-agent as a tool — see [06_multi_agent.md](06_multi_agent.md))

### R3.8 — MCP Tool Bridge
- MCP servers MUST be connectable as a tool source (see [05_mcp_support.md](05_mcp_support.md))
- All tools exposed by an MCP server MUST be auto-discoverable and usable

### R3.9 — Tool Versioning
- Tools SHOULD support a `version` field
- The registry SHOULD warn if multiple tools share the same name

---

## Tool Definition Example (Conceptual)

```python
@tool(
  description="Search the web for current information",
  timeout=15,
  needs_approval=False,
)
def search_web(query: str, max_results: int = 5) -> list[SearchResult]:
    ...
```

---

*← [Back to Index](00_INDEX.md)*
