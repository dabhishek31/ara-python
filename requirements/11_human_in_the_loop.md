# REQ-11: Human-in-the-Loop (HITL)

**Area:** I/O  
**Priority:** P1 — Should Have  
**Related:** [01_core_react_loop.md](01_core_react_loop.md) | [03_tool_system.md](03_tool_system.md) | [08_memory_state.md](08_memory_state.md) | [12_guardrails_safety.md](12_guardrails_safety.md)

---

## Overview

Human-in-the-Loop allows a running agent to pause at critical decision points,
wait for human review/approval, and then resume — without losing any state.
This is essential for high-stakes actions and regulated workflows.

---

## HITL Flow

```
Agent running...
       │
       ▼
  calls risky_tool()         ← marked needs_approval=True
       │
       ▼
  ⏸ PAUSED                   ← agent suspends here
  state serialized to store
       │
       ▼
  notification sent to human  ← Slack / Email / Dashboard
       │
       ▼
  Human reviews & decides:
   ├── APPROVE  → agent resumes with tool result
   ├── REJECT   → agent resumes with rejection message
   └── MODIFY   → human edits args, agent resumes with modified call
```

---

## Requirements

### R11.1 — Interrupt Points
- Tools marked `needs_approval=True` MUST trigger an interrupt before execution
- Interrupts MUST pause the ReAct loop cleanly between iterations

### R11.2 — State Preservation on Pause
- When paused, the agent's full state MUST be serialized and persisted
- State MUST include: conversation history, pending tool call, all prior observations
- Paused runs MUST be resumable hours or days later

### R11.3 — Interrupt Payload
- When an interrupt is raised, the SDK MUST expose:
  - `run_id` — to identify the paused run
  - `pending_tool_call` — tool name + arguments awaiting approval
  - `context_summary` — brief summary of what the agent was doing

### R11.4 — Resume API
- The SDK MUST provide a resume API:
  ```python
  # Approve
  await runner.resume(run_id, decision="approve")
  # Reject with reason
  await runner.resume(run_id, decision="reject", reason="Too risky")
  # Modify args and approve
  await runner.resume(run_id, decision="approve", modified_args={...})
  ```

### R11.5 — Notification Hook
- The SDK MUST expose an `on_interrupt(interrupt_payload)` hook
- Developers use this to send Slack messages, emails, or trigger webhooks

### R11.6 — Timeout on Pause
- A paused run MUST support a configurable `approval_timeout`
- On timeout, the run MUST either auto-cancel or auto-reject (configurable)

### R11.7 — Multi-Level Interrupts
- In multi-agent systems, interrupts in sub-agents MUST surface to the top-level orchestrator
- The orchestrator's HITL handler handles all interrupts from the full agent tree

### R11.8 — Audit Trail
- Every interrupt, decision, and resumption MUST be logged with timestamp and actor identity
- This creates a full audit trail for compliance

---

*← [Back to Index](00_INDEX.md)*
