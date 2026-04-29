# ReAct Agent SDK — Requirements Mindmap

> Master index linking all requirement modules.
> Each node below maps to a dedicated requirement file.

---

```
ReAct Agent SDK
│
├── 🔁 Core Engine
│   ├── [01] Core ReAct Loop          → 01_core_react_loop.md
│   ├── [02] Agent Lifecycle          → 02_agent_lifecycle.md
│   └── [09] LLM Provider Abstraction → 09_llm_providers.md
│
├── 🛠️ Capabilities
│   ├── [03] Tool System              → 03_tool_system.md
│   ├── [04] Skills System            → 04_skills.md
│   └── [05] MCP Support              → 05_mcp_support.md
│
├── 🤝 Multi-Agent
│   ├── [06] Multi-Agent Orchestration → 06_multi_agent.md
│   └── [07] Parallel Task Execution   → 07_parallel_execution.md
│
├── 🧠 Memory & State
│   └── [08] Memory & State Management → 08_memory_state.md
│
├── 📡 I/O
│   ├── [10] Streaming & Structured Output → 10_streaming.md
│   └── [11] Human-in-the-Loop             → 11_human_in_the_loop.md
│
├── 🛡️ Safety & Reliability
│   ├── [12] Guardrails & Safety  → 12_guardrails_safety.md
│   └── [14] Error Handling       → 14_error_handling.md
│
└── 📊 Operations
    ├── [13] Observability & Tracing → 13_observability.md
    └── [15] SDK API Design & DX     → 15_sdk_api_design.md
```

---

## Module Index

| # | Module | File | Area |
|---|--------|------|------|
| 01 | Core ReAct Loop | [01_core_react_loop.md](01_core_react_loop.md) | Engine |
| 02 | Agent Lifecycle | [02_agent_lifecycle.md](02_agent_lifecycle.md) | Engine |
| 03 | Tool System | [03_tool_system.md](03_tool_system.md) | Capabilities |
| 04 | Skills System | [04_skills.md](04_skills.md) | Capabilities |
| 05 | MCP Support | [05_mcp_support.md](05_mcp_support.md) | Capabilities |
| 06 | Multi-Agent Orchestration | [06_multi_agent.md](06_multi_agent.md) | Multi-Agent |
| 07 | Parallel Task Execution | [07_parallel_execution.md](07_parallel_execution.md) | Multi-Agent |
| 08 | Memory & State Management | [08_memory_state.md](08_memory_state.md) | Memory |
| 09 | LLM Provider Abstraction | [09_llm_providers.md](09_llm_providers.md) | Engine |
| 10 | Streaming & Structured Output | [10_streaming.md](10_streaming.md) | I/O |
| 11 | Human-in-the-Loop | [11_human_in_the_loop.md](11_human_in_the_loop.md) | I/O |
| 12 | Guardrails & Safety | [12_guardrails_safety.md](12_guardrails_safety.md) | Safety |
| 13 | Observability & Tracing | [13_observability.md](13_observability.md) | Operations |
| 14 | Error Handling & Retry | [14_error_handling.md](14_error_handling.md) | Safety |
| 15 | SDK API Design & DX | [15_sdk_api_design.md](15_sdk_api_design.md) | Operations |

| 16 | **Differentiators & The Moat** | [16_differentiators.md](16_differentiators.md) | Strategy |

---

## The Moat (Why This Exists)

> Every existing agent framework is **developer-orchestrated** — you pre-define the agent topology,
> tools, and execution pattern. The LLM navigates the map *you* drew.
>
> **This SDK makes the agent the architect.**
> The developer provides the destination. The agent draws the map and navigates it.

### The 7 Differentiators → [16_differentiators.md](16_differentiators.md)

| # | Differentiator | What makes it unique |
|---|---------------|---------------------|
| D1 | **Adaptive Strategy Selection** | Agent picks its own execution pattern (ReAct / Planner / Multi-Agent) per task — not pre-configured |
| D2 | **Dynamic Agent Spawning** | Agents create new agents at runtime — emergent topology, not pre-wired |
| D3 | **Meta-Cognition Loop** | Built-in critic monitors the agent mid-run for loops, stagnation, low confidence — and triggers reflection |
| D4 | **Goal-Oriented Input** | Developer gives a *goal* (what), SDK figures out *how* — task graph, tools, agents, execution order |
| D5 | **Composable Autonomy Levels** | 4 levels (Supervised → Background), mixable per-agent and per-tool — not binary HITL |
| D6 | **SDK as a Runtime** | Long-running agent server with task queue, agent pool, scheduling — not just a per-request library |
| D7 | **Reasoning as a Programmable Interface** | Inject into, intercept, and modify the agent's reasoning process — not a black box |

---

*Inspired by — and deliberately going beyond: Google ADK, OpenAI Agents SDK, LangGraph, CrewAI, AutoGen, Semantic Kernel, MCP Spec*

