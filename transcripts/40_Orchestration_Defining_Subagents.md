# Lecture 40: Orchestration: Defining Subagents

**Section:** Agentic Architecture  
**Lecture ID:** 58188373

---

## Transcript

### AgentDefinition class

```python
from claude_agent_sdk import ClaudeAgentOptions, AgentDefinition

research_agent = AgentDefinition(
    description="Searches the web for factual information and returns structured findings with sources. Call when the task requires gathering external information.",
    prompt="You are a research specialist. Search for the requested information and return findings in structured format with source name, URL, and publication date for each claim.",
    tools=["WebSearch", "WebFetch"],
    model="claude-haiku-4-5",   # cheaper model for sub-tasks
)

synthesis_agent = AgentDefinition(
    description="Synthesizes research findings into a coherent written report. Call after research is complete.",
    prompt="You are a technical writer. Combine the provided findings into a well-structured report, preserving source citations for every claim.",
    tools=["Write"],
    model="claude-sonnet-5",
)

options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    agents={
        "research": research_agent,
        "synthesis": synthesis_agent,
    },
    allowed_tools=["Task"],   # REQUIRED: Task is the tool that spawns subagents
)
```

---

### Description vs. system prompt

| Field | Who reads it | Purpose |
|-------|-------------|---------|
| `description` | **Parent agent** | Decides which subagent to call |
| `prompt` | **Subagent itself** | Defines its role, behavior, output format |

**Do not mix them** — role/behavior instructions in `description` do not improve subagent performance; they go in `prompt`.

---

### Key exam facts

- **`allowed_tools` must include `"Task"`** (or `"Agent"` in newer releases — both accepted) for the coordinator to spawn subagents. Missing `Task` = subagents cannot run.
- `AgentDefinition` has **no `agents` field** — subagents cannot natively spawn their own subagents.
- If `tools` and `model` are omitted from `AgentDefinition`, they inherit from the parent agent.
- The coordinator never sees the subagent's intermediate tool calls — only the final message.

---

### Tool call anatomy when spawning a subagent

When the coordinator calls a subagent, the tool block looks like:
```json
{
  "type": "tool_use",
  "name": "Task",
  "input": {
    "subagent_type": "research",
    "prompt": "Find adoption statistics for..."
  }
}
```

**Avoid adding `Task` to a subagent's `allowed_tools`** — this would break the hub-and-spoke design.
