# REQ-10: Streaming & Structured Output

**Area:** I/O  
**Priority:** P1 — Should Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [09_llm_providers.md](09_llm_providers.md) | [13_observability.md](13_observability.md)

---

## Overview

The SDK MUST support real-time streaming of agent output and enforce structured
(schema-validated) responses when needed. Streaming reduces perceived latency.
Structured output enables reliable downstream processing.

---

## Event Stream Model

```
Agent run emits a stream of typed events:

  RunStarted
  IterationStarted (iteration=1)
    ThoughtGenerated  ← LLM reasoning text
    ToolCallRequested ← tool name + args
    ToolCallCompleted ← tool result
  IterationStarted (iteration=2)
    ThoughtGenerated
    FinalAnswerStarted
    TextDelta  TextDelta  TextDelta  ← streaming tokens
  FinalAnswerCompleted
  RunCompleted
```

---

## Requirements

### R10.1 — Streaming API
- The SDK MUST expose a streaming run API:
  ```python
  async for event in agent.run_stream(input):
      handle(event)
  ```
- Streaming MUST work for the final answer text (token by token)

### R10.2 — Typed Event System
- All events MUST be strongly typed with a discriminated union
- Event types MUST include at minimum:
  - `RunStartedEvent`
  - `IterationStartedEvent`
  - `ThoughtEvent`
  - `ToolCallStartedEvent`
  - `ToolCallCompletedEvent`
  - `TextDeltaEvent` (streaming token)
  - `FinalAnswerEvent`
  - `RunCompletedEvent`
  - `RunFailedEvent`

### R10.3 — Non-Streaming (Awaitable) API
- The SDK MUST also expose a simple non-streaming API for convenience:
  ```python
  result = await agent.run(input)
  ```

### R10.4 — Structured Output
- An agent MUST be configurable to return a structured response (Pydantic model / JSON Schema)
- When `output_schema` is set, the LLM MUST be forced to output valid JSON matching the schema
- The SDK MUST validate the output and retry if the LLM returns invalid JSON

### R10.5 — Structured Output + Tools Coexistence
- When structured output is required AND tools are available:
  - The agent MUST be able to use tools freely during reasoning
  - Only the FINAL answer is forced into the structured schema

### R10.6 — Streaming Interruption
- Streaming MUST be interruptible (e.g., user cancels mid-stream)
- Partial results MUST be returned safely without corrupting state

### R10.7 — SSE / WebSocket Transport
- For web integrations, the SDK MUST support serving the event stream over:
  - Server-Sent Events (SSE)
  - WebSocket
- This enables real-time agent output in browser UIs

---

## Structured Output Example (Conceptual)

```python
class ResearchReport(BaseModel):
    title: str
    summary: str
    sources: list[str]
    confidence: float

result: ResearchReport = await agent.run(
    "Research quantum computing breakthroughs in 2024",
    output_schema=ResearchReport,
)
```

---

*← [Back to Index](00_INDEX.md)*
