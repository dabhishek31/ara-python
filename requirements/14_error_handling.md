# REQ-14: Error Handling & Retry

**Area:** Safety  
**Priority:** P0 — Must Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [09_llm_providers.md](09_llm_providers.md) | [12_guardrails_safety.md](12_guardrails_safety.md) | [13_observability.md](13_observability.md)

---

## Overview

Agent systems are inherently unreliable — LLMs hallucinate, APIs fail, tools timeout.
The SDK MUST handle failures gracefully at every level, with intelligent retry logic,
circuit breakers, and clean error surfacing to the agent and end user.

---

## Error Categories

```
┌──────────────────────────────────────────────┐
│  TRANSIENT (retry)                           │
│  • Rate limit (429)                          │
│  • Timeout (408, 504)                        │
│  • Temporary server error (500, 503)         │
└──────────────────────────────────────────────┘
┌──────────────────────────────────────────────┐
│  PERMANENT (fail fast, do not retry)         │
│  • Auth error (401, 403)                     │
│  • Invalid request (400)                     │
│  • Schema validation failure                 │
│  • Guardrail block                           │
└──────────────────────────────────────────────┘
┌──────────────────────────────────────────────┐
│  AGENT-LEVEL (LLM can reason about it)       │
│  • Tool execution error                      │
│  • Tool timeout                              │
│  • Malformed tool arguments                  │
└──────────────────────────────────────────────┘
```

---

## Requirements

### R14.1 — Tool Error as Observation
- When a tool fails, the error MUST be returned to the LLM as an observation
- The LLM MUST be able to reason about the failure and try a different strategy
- Agent-level errors MUST NOT crash the run

### R14.2 — Retry with Exponential Backoff
- Transient LLM API errors MUST be retried automatically
- Retry policy: exponential backoff with jitter
  - Attempt 1: immediate
  - Attempt 2: 1s delay
  - Attempt 3: 2s delay
  - Attempt 4: 4s delay
  - Max attempts: configurable (default: 3)

### R14.3 — LLM Fallback
- If the primary LLM fails after all retries, the SDK MUST try the fallback model (see [09_llm_providers.md](09_llm_providers.md))

### R14.4 — Circuit Breaker
- The SDK MUST implement a circuit breaker per LLM provider and per tool
- Circuit opens after N consecutive failures within a time window
- While open: fail fast (do not attempt the call)
- After a cooldown, circuit moves to half-open (try one request to test recovery)

### R14.5 — Max Iterations Exceeded
- When `max_iterations` is reached, raise `MaxIterationsError`
- Include the partial run result in the error for debugging

### R14.6 — Structured SDK Errors
- The SDK MUST define a clear error hierarchy:
  ```
  AgentSDKError
  ├── ConfigurationError
  ├── LLMProviderError
  │   ├── RateLimitError
  │   ├── AuthenticationError
  │   └── ModelError
  ├── ToolError
  │   ├── ToolTimeoutError
  │   ├── ToolValidationError
  │   └── ToolExecutionError
  ├── GuardrailError
  ├── MaxIterationsError
  └── AgentCancelledError
  ```

### R14.7 — Error Callbacks
- The SDK MUST expose `on_tool_error`, `on_llm_error`, `on_run_error` hooks
- These allow custom logging, alerting, or fallback behavior

### R14.8 — Full Error Tracing
- All errors MUST be captured in the run trace with:
  - Error type, message, stack trace
  - The iteration and context in which it occurred

---

*← [Back to Index](00_INDEX.md)*
