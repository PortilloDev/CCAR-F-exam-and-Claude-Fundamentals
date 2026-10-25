# Lecture 30: Agent SDK: The Loop as a Library

**Section:** Agentic Architecture  
**Lecture ID:** 58188321

---

## Transcript

### Three interaction layers

| Layer | What it is |
|-------|-----------|
| **Client SDK** | Library for the raw Claude Messages API (stateless, no loop) |
| **Agent SDK** | Library that adds: agentic loop, built-in tools, permission control, hooks, session management |
| **Claude Code** | Black-box product (terminal/desktop app); uses Agent SDK internally |

---

### Agent SDK vs. building your own loop

**Client SDK (manual loop):** You call `messages.create`, check `stop_reason`, dispatch tools, append results, repeat.

**Agent SDK:** The `query()` method handles all of that. You pass a prompt + options; it runs the full loop and streams results.

---

### Creating an agent

```python
from claude_agent_sdk import ClaudeAgentOptions, query

options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    system_prompt="You are a helpful assistant.",
    allowed_tools=["Read", "Grep"],
    # max_turns, max_budget, output_format, etc.
)

async for chunk in query(prompt="Summarize the auth module", options=options):
    if chunk is ToolUseBlock:
        print("Tool call:", chunk)
```

---

### Key `ClaudeAgentOptions` fields

| Field | Purpose |
|-------|---------|
| `system_prompt` | Always loaded at the top of context |
| `agents` | Sub-agents this agent can orchestrate (AgentDefinition objects) |
| `mcp_servers` | MCP servers available as tools |
| `setting_sources` | Where to load `CLAUDE.md` and skill files from |
| `allowed_tools` | Tools that run without a permission prompt |
| `permission_mode` | Default behavior for tools not in `allowed_tools` |
| `hooks` | Code to run before/after tool calls |
| `max_turns` | Cap on loop iterations |
| `max_budget` | Cap in USD |
| `output_format` | Structured output schema (JSON schema) |

**Exam note:** Preferred structured output approach = use `tool_choice` with a tool whose input schema is your expected output format (not the `output_format` field directly).

---

### Result message subtypes

| Subtype | Meaning |
|---------|---------|
| `success` | Loop completed normally |
| `max_turns` | Hit the turn limit |
| `max_budget` | Hit the cost cap |
| `error` | Runtime error |
| `structured_output_failed` | Conversion to structured output failed after retries |
