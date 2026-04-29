# REQ-01: Core ReAct Loop

**Area:** Engine  
**Priority:** P0 — Must Have  
**Related:** [02_agent_lifecycle.md](02_agent_lifecycle.md) | [03_tool_system.md](03_tool_system.md) | [14_error_handling.md](14_error_handling.md)

---

## Overview

The ReAct (Reasoning + Acting) loop is the fundamental execution primitive of the SDK.
Every agent run operates in an iterative cycle: **Think → Act → Observe → Repeat**.

---

## The Loop Steps

```
User Input
    │
    ▼
┌───────────────────────────────────────────────────────┐
│                       THINK                           │
│  LLM receives: system prompt + full history + tools   │
│  LLM reasons, plans, and decides what to do next      │
│                                                       │
│  LLM emits one of three signals:                      │
│  (A) tool_call    → needs to act on something         │
│  (B) continue     → needs to think/plan more          │
│  (C) stop_reason  → task is complete, give answer     │
└───────────┬───────────────────┬───────────────────────┘
            │                   │                   │
           (A)                 (B)                 (C)
     tool_call emitted    no tool call,         stop_reason
            │             keep thinking            emitted
            ▼                   │                   │
    ┌──────────────┐            │                   ▼
    │     ACT      │            │             ┌──────────┐
    │  Execute the │            │             │   DONE   │
    │  tool call   │            │             │  Return  │
    │  (sync/async)│            │             │  Final   │
    └──────┬───────┘            │             │  Answer  │
           │                   │             └──────────┘
           ▼                   │
    ┌──────────────┐            │
    │   OBSERVE    │            │
    │  Inject tool │            │
    │  result (or  │            │
    │  error) back │            │
    │  into history│            │
    └──────┬───────┘            │
           │                   │
           └────────────────────┘
                   │
                   ▼
              back to THINK
    (with updated history & observations)
```

> **Key Insight:** The loop exit is **entirely LLM-driven**.
> The LLM signals it is done by emitting `finish_reason = stop` (or `end_turn`).
> A missing tool call does **NOT** mean the agent is done —
> the LLM can iterate with pure reasoning/planning steps before calling a tool or stopping.

---

## Requirements

### R1.1 — LLM-Driven Loop Control
- The loop exit is **entirely controlled by the LLM**, not by the SDK logic
- On every iteration, the LLM MUST emit one of three signals:
  - **`tool_call`** — LLM wants to execute a tool; loop continues after observation
  - **`continue`** — LLM wants another reasoning/planning step with no tool call; loop continues
  - **`stop` (finish_reason = stop / end_turn)** — LLM is satisfied; loop exits and final answer is returned
- The SDK MUST NOT exit the loop just because the LLM did not emit a tool call

### R1.1b — Pure-Thinking Iterations (No Tool Call)
- When the LLM emits a `continue` signal (no tool call, not yet done):
  - The current reasoning output MUST be appended to the conversation history
  - The loop MUST call the LLM again immediately with the updated history
  - This allows the LLM to: break down a task, plan sub-steps, self-reflect, or build up reasoning before acting
- Example use cases: chain-of-thought reasoning, task decomposition, self-critique, multi-step planning without external calls

### R1.2 — Hard Exit Conditions (SDK-enforced)
- The loop MUST force-exit when any of these conditions are met, regardless of LLM signal:
  - `max_iterations` limit is reached → raise `MaxIterationsError`
  - Token budget is exhausted → raise `TokenBudgetError`
  - A fatal tool or LLM error occurs (non-retriable)
  - A guardrail blocks the response
  - A Human-in-the-Loop interrupt is raised → pause, do not exit

### R1.3 — Thought Capture
- Every LLM response (whether `tool_call`, `continue`, or `stop`) MUST be captured in the run trace
- The reasoning text MUST be stored per-iteration even if not surfaced to the end-user

### R1.4 — Action Dispatch
- When the LLM emits a `tool_call`, the loop MUST:
  1. Parse the tool name and arguments from the LLM response
  2. Validate arguments against the tool's registered schema
  3. Execute the tool (sync or async, with timeout)
  4. Capture the result OR the error as a structured observation

### R1.5 — Observation Injection
- Tool results MUST be appended to the conversation history as an `observation` message
- Both success results AND errors MUST be returned to the LLM so it can reason about failures
- The LLM MUST be allowed to try a different tool or approach after seeing an error

### R1.6 — Max Iterations Guard
- `max_iterations` MUST be a configurable parameter per agent (default: 10)
- When limit is hit, agent MUST gracefully stop and return best available result so far
- An `MaxIterationsError` with the partial result MUST be raised

### R1.7 — Token Budget (Optional)
- The SDK SHOULD support an optional `max_tokens_per_run` budget
- When the cumulative token count approaches the limit, the loop SHOULD attempt to finalize
- See token tracking in [09_llm_providers.md](09_llm_providers.md)

### R1.8 — Sync & Async Support
- The loop MUST support both synchronous and asynchronous execution modes

---

## Inspired By

| Framework | How they implement it |
|-----------|----------------------|
| Google ADK | `LlmAgent` with built-in Thought→Tool→Observation cycle |
| LangGraph | Graph nodes: `agent_node → tool_node → agent_node` with conditional edges |
| OpenAI Agents SDK | `Runner` class managing the while-loop with `RunState` |
| CrewAI | Task executor with sequential/hierarchical process |

---

*← [Back to Index](00_INDEX.md)*
