# REQ-15: SDK API Design & Developer Experience (DX)

**Area:** Operations  
**Priority:** P0 — Must Have  
**Related:** [02_agent_lifecycle.md](02_agent_lifecycle.md) | [03_tool_system.md](03_tool_system.md) | [10_streaming.md](10_streaming.md) | [13_observability.md](13_observability.md)

---

## Overview

The SDK's public API must be clean, intuitive, and minimal.
A developer MUST be able to build a working agent in under 10 lines of code,
while the SDK scales to support production-grade complex systems.

---

## Design Philosophy

> **"Simple things simple, complex things possible."**
> - Zero-config defaults that work out of the box
> - Progressive disclosure of advanced features
> - Type-safe everywhere (Pydantic + full type hints)
> - Async-first, sync-compatible

---

## Requirements

### R15.1 — Minimal "Hello World" API
A basic agent MUST work in ≤ 10 lines:
```python
from agent_sdk import Agent, tool

@tool(description="Search the web")
def search(query: str) -> str: ...

agent = Agent(
    name="assistant",
    instructions="You are a helpful assistant.",
    model="gemini-2.0-flash",
    tools=[search],
)

result = await agent.run("What is the weather in Mumbai?")
print(result.output)
```

### R15.2 — Type Safety
- All SDK classes MUST have full type annotations
- Pydantic v2 MUST be used for all config, input, and output models
- The SDK MUST be mypy and pyright compatible

### R15.3 — Async-First
- All I/O-bound operations (LLM calls, tool calls, memory) MUST be `async`
- Sync wrappers MUST be provided for environments without async support:
  ```python
  result = agent.run_sync("What is 2+2?")
  ```

### R15.4 — Fluent Builder API (Optional)
- Advanced config SHOULD be expressible via a fluent builder pattern for readability

### R15.5 — CLI Tool
- The SDK MUST ship a CLI for common operations:
  ```bash
  agent-sdk run --agent my_agent.py --input "Hello"
  agent-sdk trace --run-id abc-123
  agent-sdk test --suite tests/agent_tests.yaml
  ```

### R15.6 — Evaluation Framework
- The SDK MUST include a built-in eval harness for testing agent quality:
  - Define test cases (input → expected output criteria)
  - Run the agent and score using LLM-as-judge or exact match
  - Generate evaluation reports

### R15.7 — Local Dev Server
- The SDK MUST ship a local dev playground:
  ```bash
  agent-sdk dev
  ```
  - Opens a web UI to chat with your agent
  - Shows real-time trace panel alongside the conversation
  - No external dependencies required

### R15.8 — Versioning & Stability
- The SDK MUST follow Semantic Versioning (semver)
- Public API MUST be stable across minor versions
- Breaking changes only in major versions, with migration guides

### R15.9 — Documentation
- Every public class, method, and parameter MUST have docstrings
- Auto-generated API reference (Sphinx / MkDocs)
- Quickstart guide, cookbook examples, and real-world tutorials

### R15.10 — Package & Distribution
- SDK MUST be published to PyPI: `pip install agent-sdk`
- Minimal dependencies (not 200-package installs)
- Optional extras: `pip install agent-sdk[mcp,langfuse,redis]`

---

## SDK Package Structure (Conceptual)

```
agent_sdk/
├── core/
│   ├── agent.py          # Agent class
│   ├── runner.py         # ReAct loop runner
│   └── context.py        # Run context
├── tools/
│   ├── base.py           # @tool decorator
│   └── builtins/         # search, code_exec, http, etc.
├── skills/
│   ├── base.py           # BaseSkill
│   └── builtins/         # WebResearch, CodeGen, etc.
├── memory/
│   ├── working.py
│   ├── session.py
│   └── longterm/         # redis, postgres, etc.
├── providers/
│   ├── base.py           # LLMProvider interface
│   ├── openai.py
│   ├── gemini.py
│   ├── anthropic.py
│   └── ollama.py
├── mcp/
│   └── client.py         # MCP client integration
├── guardrails/
│   └── base.py
├── observability/
│   └── tracer.py
└── cli/
    └── main.py
```

---

*← [Back to Index](00_INDEX.md)*
