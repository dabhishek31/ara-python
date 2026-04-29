# REQ-08: Memory & State Management

**Area:** Memory  
**Priority:** P0 — Must Have  
**Related:** [02_agent_lifecycle.md](02_agent_lifecycle.md) | [06_multi_agent.md](06_multi_agent.md) | [11_human_in_the_loop.md](11_human_in_the_loop.md)

---

## Overview

Memory gives agents the ability to remember context — both within a single run
and across multiple sessions. State management ensures the agent's execution
can be checkpointed, paused, and resumed reliably.

---

## Memory Layers

```
┌─────────────────────────────────────────────┐
│  Working Memory (in-context)                │
│  → Current conversation turns & tool results│
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Session Memory (short-term, per-session)   │
│  → Facts learned during this conversation   │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Long-term Memory (persistent, cross-session)│
│  → User preferences, past decisions, history│
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Knowledge Memory (RAG / vector store)      │
│  → External documents, databases, indexes   │
└─────────────────────────────────────────────┘
```

---

## Requirements

### R8.1 — Working Memory (Conversation History)
- Every agent run MUST maintain an ordered list of messages (user, assistant, tool results)
- This is the "context window" fed to the LLM on every iteration
- History MUST be truncatable (sliding window or summarization) when it exceeds token limits

### R8.2 — Session Memory
- The SDK MUST support saving key facts extracted during a session
- Facts MUST be injectable back into the system prompt on future turns
- Session memory MUST be scoped to `session_id`

### R8.3 — Long-term (Persistent) Memory
- The SDK MUST support a pluggable long-term memory backend:
  - In-memory (default, for dev)
  - Redis / DynamoDB / Postgres (for production)
- Long-term memory MUST survive agent restarts and be queryable by `user_id`

### R8.4 — RAG / Knowledge Memory
- The SDK MUST support connecting a vector store for semantic retrieval
- On each iteration (or on demand), relevant context MUST be fetched and injected
- Supported backends: Pinecone, Chroma, Weaviate, pgvector (pluggable interface)

### R8.5 — State Checkpointing
- The agent's full state (conversation, tool history, variables) MUST be serializable to JSON
- Checkpoints MUST be saveable after every iteration
- A run MUST be resumable from any checkpoint

### R8.6 — Shared State (Multi-Agent)
- When multiple agents run in a system, a shared state object MUST be optionally available
- Agents MUST explicitly read/write to shared state — no implicit coupling

### R8.7 — State Schema
- Developers MUST be able to define a typed state schema (Pydantic model)
- The SDK enforces that all state reads/writes conform to the schema

---

## Memory Config Example (Conceptual)

```python
MemoryConfig(
  working=True,                    # always on
  session=SessionMemory(),         # in-process
  long_term=RedisMemory(url="..."),
  knowledge=ChromaVectorStore(collection="docs"),
)
```

---

*← [Back to Index](00_INDEX.md)*
