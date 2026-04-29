# REQ-02: Agent Lifecycle

**Area:** Engine  
**Priority:** P0 — Must Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [08_memory_state.md](08_memory_state.md) | [13_observability.md](13_observability.md)

---

## Overview

Defines how an Agent is created, configured, run, paused, resumed, and destroyed.
The lifecycle covers a single agent run from initialization to final output.

---

## Lifecycle States

```
CREATED → INITIALIZED → RUNNING → (PAUSED) → COMPLETED
                                 ↘           ↗
                                  → FAILED →
                                  → CANCELLED
```

---

## Requirements

### R2.1 — Agent Definition
- An agent MUST be definable with:
  - `name` — unique identifier
  - `instructions` / `system_prompt` — the agent's role and behavior
  - `model` — which LLM to use (see [09_llm_providers.md](09_llm_providers.md))
  - `tools` — list of tools available (see [03_tool_system.md](03_tool_system.md))
  - `skills` — list of skills to load (see [04_skills.md](04_skills.md))
  - `max_iterations` — loop limit
  - `memory` — memory config (see [08_memory_state.md](08_memory_state.md))

### R2.2 — Run Context
- Each run MUST have a unique `run_id`
- Each run MUST carry a `session_id` linking it to a user/conversation
- Run context MUST be passed through the entire lifecycle

### R2.3 — Initialization Hooks
- The SDK MUST support `on_start` hook called before the first iteration
- Useful for loading memory, validating config, warming up connections

### R2.4 — Completion Hooks
- The SDK MUST support `on_complete(result)` and `on_error(error)` hooks
- These run after the loop exits for any reason

### R2.5 — Cancellation
- A running agent MUST be cancellable mid-execution
- Cancellation MUST cleanly stop the loop and call `on_cancel` hook

### R2.6 — Agent Cloning / Templates
- Agents SHOULD be cloneable so one definition can spawn many parallel instances
- Template agents MUST support per-run override of `instructions` and `model`

### R2.7 — Stateless vs Stateful Runs
- Agents MUST support **stateless** (no session memory) runs for simple one-shot tasks
- Agents MUST support **stateful** runs with persistent session/memory

---

## Agent Config Schema (Conceptual)

```python
Agent(
  name="research-agent",
  instructions="You are a research assistant...",
  model="gemini-2.0-flash",
  tools=[search_tool, calculator_tool],
  skills=["web-research"],
  max_iterations=15,
  memory=MemoryConfig(type="session"),
)
```

---

*← [Back to Index](00_INDEX.md)*
