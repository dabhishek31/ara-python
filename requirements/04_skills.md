# REQ-04: Skills System

**Area:** Capabilities  
**Priority:** P1 — Should Have  
**Related:** [02_agent_lifecycle.md](02_agent_lifecycle.md) | [03_tool_system.md](03_tool_system.md) | [08_memory_state.md](08_memory_state.md)

---

## Overview

Skills are **reusable, packaged capabilities** that can be attached to any agent.
Unlike Tools (which are single callable functions), a Skill is a **bundle** that may include:
- A set of related tools
- Domain-specific system prompt additions
- Reference knowledge / context injections
- Pre-configured behaviors

Think of a Skill as a "plugin" or "capability pack" that gives an agent expertise in a domain.

---

## Skills vs Tools

| Aspect | Tool | Skill |
|--------|------|-------|
| Scope | Single function | Bundle of related capabilities |
| Changes LLM behavior? | No (only adds actions) | Yes (also adds context/instructions) |
| Reusability | Per-agent | Cross-agent, plug-and-play |
| Example | `search_web()` | `WebResearchSkill` (search + scrape + summarize + instructions) |

---

## Requirements

### R4.1 — Skill Definition
- A Skill MUST be definable as a class or declarative config
- A Skill MUST declare:
  - `name` — unique identifier
  - `description` — what capability this adds
  - `tools` — list of tools this skill provides
  - `system_prompt_addon` — additional instructions injected into agent's system prompt
  - `knowledge` — optional static reference docs / context

### R4.2 — Skill Registration
- Skills MUST be registerable in a global `SkillRegistry`
- An agent MUST be able to load skills by name: `skills=["web-research", "code-gen"]`

### R4.3 — Skill Composition
- An agent MUST support multiple skills simultaneously
- Skills MUST NOT conflict — their tools and prompt addons are merged cleanly

### R4.4 — Skill Isolation
- Skills SHOULD be self-contained — no hidden global state
- Skills MUST be shareable across agents in the same multi-agent system

### R4.5 — Built-in Skills (Shipped with SDK)
The SDK SHOULD ship with common skills out of the box:
- `WebResearchSkill` — search + read + summarize web pages
- `CodeExecutionSkill` — write + run + debug code
- `DataAnalysisSkill` — read CSV/JSON, run calculations, plot charts
- `FileSystemSkill` — read, write, list files safely
- `CalendarSkill` — interact with calendar APIs
- `MemorySkill` — explicit memory read/write operations

### R4.6 — Custom Skills
- Developers MUST be able to create and publish custom skills as packages
- The SDK MUST provide a `BaseSkill` class to extend

### R4.7 — MCP as a Skill Source
- An MCP server connection SHOULD be wrappable as a Skill
- All tools from the MCP server become available as part of that skill (see [05_mcp_support.md](05_mcp_support.md))

---

## Skill Definition Example (Conceptual)

```python
class WebResearchSkill(BaseSkill):
    name = "web-research"
    description = "Enables web search and page reading"
    tools = [search_web, read_page, summarize_page]
    system_prompt_addon = """
      When researching, always verify information from at least 2 sources.
      Cite your sources in the final answer.
    """
```

---

*← [Back to Index](00_INDEX.md)*
