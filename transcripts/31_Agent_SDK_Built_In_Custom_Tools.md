# Lecture 31: Agent SDK: Built-In & Custom Tools

**Section:** Agentic Architecture  
**Lecture ID:** 58188333

---

## Transcript

### Built-in tools (Agent SDK)

| Tool | Purpose | Requires permission? |
|------|---------|---------------------|
| **Read** | Read a full file | Yes |
| **Glob** | Find files by name/path pattern | Yes |
| **Grep** | Search content inside files | Yes |
| **Agent** | Call a sub-agent | Yes |
| **Write** | Replace a full file | Yes |
| **Edit** | Change a portion of a file | Yes |
| **Bash** | Run shell scripts | Yes |
| **WebSearch** | Google-like web search | Yes |
| **WebFetch** | Load a specific URL | Yes |

All built-in tools require permission approval (unless pre-approved in `allowed_tools`). Custom tools also always require permission.

---

### Assigning built-in tools

```python
# Only Read, Glob, Grep — explicit list
options = ClaudeAgentOptions(
    tools=["Read", "Glob", "Grep"],
    cwd="/path/to/project",
)

# No built-in tools — add your own server only
options = ClaudeAgentOptions(
    tools=[],
    mcp_servers=[my_local_server],
)

# Match Claude Code's default tool set
options = ClaudeAgentOptions(
    tools=[{"type": "preset", "preset": "claude_code"}],
    cwd="/path/to/project",
)
```

---

### Creating a custom tool

```python
from claude_agent_sdk import tool

@tool(
    name="get_order",
    description="Retrieves order details by order ID. Use when the user asks about a specific order.",
    annotations={"readOnlyHint": True},  # enables parallel execution with other read-only tools
)
def get_order(order_id: str) -> dict:
    try:
        result = db.fetch_order(order_id)
        return {"type": "text", "text": str(result)}
    except Exception as e:
        return {"is_error": True, "content": [{"type": "text", "text": str(e)}]}
```

**`readOnlyHint=True`:** Tells the SDK the tool is safe to run in parallel with other read-only tools — performance optimization.

---

### Exposing a custom tool as an in-process MCP server

```python
from claude_agent_sdk import create_sdk_mcp_server

my_server = create_sdk_mcp_server(
    name="order-tools",
    version="1.0.0",
    tools=[get_order],
)

options = ClaudeAgentOptions(
    mcp_servers=[my_server],
    allowed_tools=["mcp.order-tools.get_order"],
)
```

---

### When to use each approach

| Approach | When |
|----------|------|
| In-process `@tool` + `create_sdk_mcp_server` | Small SDK app, tool only needed here |
| External hosted MCP server | Shared across apps or Claude Code sessions |
| Client SDK `@tool` annotation | Custom loop, maximum control |
