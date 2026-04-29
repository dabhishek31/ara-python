# REQ-09: LLM Provider Abstraction

**Area:** Engine  
**Priority:** P0 — Must Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [02_agent_lifecycle.md](02_agent_lifecycle.md) | [10_streaming.md](10_streaming.md)

---

## Overview

The SDK MUST be model-agnostic. Agent logic and tool definitions MUST work
identically regardless of which LLM is powering the agent. Swapping models
should require changing only one config value.

---

## Provider Abstraction Layer

```
Agent Logic (model-agnostic)
       │
       ▼
  LLM Interface (SDK abstraction)
       │
  ┌────┼────────────────────────────┐
  ▼    ▼                            ▼
OpenAI  Google Gemini   Anthropic   Ollama (local)
GPT-4o  Gemini-2.0      Claude 3.5  Llama 3
```

---

## Requirements

### R9.1 — Unified LLM Interface
- The SDK MUST define a single `LLMProvider` interface that all model integrations implement
- The interface MUST standardize:
  - `generate(messages, tools, config) → LLMResponse`
  - `stream(messages, tools, config) → AsyncIterator[LLMEvent]`
  - `count_tokens(messages) → int`

### R9.2 — Built-in Providers
The SDK MUST ship integrations for:
- **OpenAI** — GPT-4o, GPT-4o-mini, o1, o3
- **Anthropic** — Claude 3.5 Sonnet, Claude 3 Haiku
- **Google** — Gemini 2.0 Flash, Gemini 1.5 Pro
- **Ollama** — any local model (Llama, Mistral, Phi, etc.)
- **LiteLLM** — as a universal fallback adapter for 100+ models

### R9.3 — Model Config Per Agent
- Each agent MUST specify its model independently:
  ```python
  Agent(model="gemini-2.0-flash", ...)
  Agent(model="gpt-4o", ...)
  ```
- Model config MUST support:
  - `temperature`, `top_p`, `max_tokens`
  - `stop_sequences`
  - Provider-specific extra params via `extra_kwargs`

### R9.4 — Tool Call Normalization
- Different LLMs have different tool-calling formats
- The SDK MUST normalize all tool call formats into a single internal representation
- Tool definitions (JSON Schema) MUST be translated to each provider's native format automatically

### R9.5 — Fallback / Retry on Provider Error
- The SDK MUST support defining a `fallback_model` per agent
- If the primary model fails (rate limit, timeout), the SDK MUST automatically retry with the fallback
- See also: [14_error_handling.md](14_error_handling.md)

### R9.6 — Token Budget Tracking
- The SDK MUST track tokens used (input + output) per iteration and per run
- Token counts MUST be exposed in the run result and trace

### R9.7 — Model Routing (Advanced)
- The SDK SHOULD support a `ModelRouter` that selects the model dynamically based on:
  - Task type (e.g., use cheap model for simple steps, powerful model for hard reasoning)
  - Cost budget
  - Latency requirements

---

## Provider Config Example (Conceptual)

```python
# Simple
Agent(model="gpt-4o")

# Full config
Agent(
  model=ModelConfig(
    provider="openai",
    model_id="gpt-4o",
    temperature=0.2,
    max_tokens=4096,
    fallback="gemini-2.0-flash",
  )
)
```

---

*← [Back to Index](00_INDEX.md)*
