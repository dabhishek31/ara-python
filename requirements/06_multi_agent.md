# REQ-06: Multi-Agent Orchestration

**Area:** Multi-Agent  
**Priority:** P1 — Should Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [02_agent_lifecycle.md](02_agent_lifecycle.md) | [07_parallel_execution.md](07_parallel_execution.md) | [08_memory_state.md](08_memory_state.md)

---

## Overview

The SDK MUST enable multiple agents to collaborate on complex tasks.
Agents can delegate to each other, run in hierarchies, or work as peer networks.

---

## Collaboration Patterns

### Pattern 1: Orchestrator → Worker (Hierarchical)
```
Orchestrator Agent
  ├── delegates to → Research Agent
  ├── delegates to → Writing Agent
  └── delegates to → QA Agent
```

### Pattern 2: Handoff (Sequential Transfer)
```
Triage Agent → (handoff) → Billing Agent → (handoff) → Support Agent
```

### Pattern 3: Agent-as-Tool
```
Manager Agent
  ├── calls research_agent() as a tool → gets result
  └── calls analyst_agent() as a tool  → gets result
  → synthesizes final answer
```

---

## Requirements

### R6.1 — Agent-as-Tool
- Any agent MUST be wrappable as a tool callable by another agent
- The calling agent treats the sub-agent as a black box that takes input and returns output
- Sub-agent runs independently with its own ReAct loop

### R6.2 — Handoffs
- An agent MUST be able to transfer full conversation control to another agent
- Handoffs MUST carry the conversation history and context to the receiving agent
- Handoff is modeled as a special tool call: `transfer_to_<agent_name>`

### R6.3 — Orchestrator Pattern
- The SDK MUST support defining an **orchestrator** agent that:
  - Receives the top-level goal
  - Plans and delegates subtasks to worker agents
  - Collects results and synthesizes a final answer

### R6.4 — Shared Context
- Agents in the same multi-agent system SHOULD be able to share a common context object
- Shared context allows agents to read state written by other agents

### R6.5 — Agent Registry
- The SDK MUST maintain a registry of named agents within a run
- Agents MUST be resolvable by name for dynamic delegation

### R6.6 — Delegation Depth Limit
- The SDK MUST enforce a configurable `max_delegation_depth` to prevent infinite loops
- Default: 5 levels deep

### R6.7 — Isolation
- Each agent in a multi-agent system MUST have its own memory scope by default
- Shared memory MUST be explicitly opted into (see [08_memory_state.md](08_memory_state.md))

---

## Inspired By

| Framework | Multi-Agent Feature |
|-----------|-------------------|
| Google ADK | `AgentTool` wraps sub-agents; hierarchical delegation |
| OpenAI Agents SDK | `handoffs` param + `agent.as_tool()` |
| CrewAI | `Crew` with sequential/hierarchical `Process` |
| AutoGen | `GroupChat` with `Speaker` selection strategy |
| LangGraph | Supervisor + Worker graph pattern |

---

*← [Back to Index](00_INDEX.md)*
