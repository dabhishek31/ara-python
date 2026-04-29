# REQ-12: Guardrails & Safety

**Area:** Safety  
**Priority:** P0 — Must Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [11_human_in_the_loop.md](11_human_in_the_loop.md) | [14_error_handling.md](14_error_handling.md)

---

## Overview

Guardrails are safety layers that validate inputs, agent behavior, and outputs
to prevent harmful, biased, or policy-violating responses.
They operate at multiple points in the agent lifecycle.

---

## Guardrail Touch Points

```
User Input
    │
    ▼
[INPUT GUARDRAIL]     ← validate before LLM sees it
    │
    ▼
 ReAct Loop
    │
    ▼
[TOOL CALL GUARDRAIL] ← validate before tool executes
    │
    ▼
  Tool Result
    │
    ▼
[OUTPUT GUARDRAIL]    ← validate before sending to user
    │
    ▼
 Final Response → User
```

---

## Requirements

### R12.1 — Input Guardrails
- The SDK MUST support pluggable input validators that run before the LLM is called
- Default checks MUST include:
  - Prompt injection detection
  - PII detection (emails, phone numbers, SSNs)
  - Toxicity / hate speech detection
  - Jailbreak attempt detection

### R12.2 — Output Guardrails
- The SDK MUST support pluggable output validators that run on the final LLM response
- Default checks MUST include:
  - PII leakage detection
  - Hallucination flags (if fact-checking is enabled)
  - Policy compliance check

### R12.3 — Tool Call Guardrails
- Before executing any tool, the SDK MUST run configurable pre-execution checks
- Examples: block certain URLs, disallow file writes outside a sandbox directory

### R12.4 — Guardrail Actions
When a guardrail is triggered, it MUST define one of these actions:
- `block` — stop execution, return error to user
- `warn` — log warning but continue
- `modify` — sanitize input/output and continue
- `interrupt` — pause for human review (see [11_human_in_the_loop.md](11_human_in_the_loop.md))

### R12.5 — Pluggable Guardrail Interface
- Developers MUST be able to register custom guardrails:
  ```python
  @guardrail(on="input", action="block")
  def no_competitor_mentions(text: str) -> GuardrailResult:
      ...
  ```

### R12.6 — Excessive Agency Prevention
- The SDK MUST enforce a principle of least privilege for tools
- Agent MUST only have access to tools explicitly granted to it
- No agent MUST be able to grant itself new tools at runtime

### R12.7 — System Prompt Protection
- The system prompt MUST NOT be extractable via prompt injection attacks
- The SDK MUST detect and block attempts to reveal or override the system prompt

### R12.8 — Max Iteration Safety
- As a safety guardrail, `max_iterations` enforcement is non-negotiable
- No configuration MUST be able to set `max_iterations` to unlimited

---

## Inspired By

| Framework | Safety Feature |
|-----------|---------------|
| OpenAI Agents SDK | Built-in `input_guardrails` + `output_guardrails` on Agent |
| Google ADK | Safety settings per model call; guardrail callbacks |
| OWASP LLM Top 10 | Prompt injection, excessive agency, sensitive data exposure |

---

*← [Back to Index](00_INDEX.md)*
