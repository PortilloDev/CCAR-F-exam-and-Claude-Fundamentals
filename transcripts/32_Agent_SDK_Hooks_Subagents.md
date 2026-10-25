# Lecture 32: Agent SDK: Hooks & Subagents

**Section:** Agentic Architecture  
**Lecture ID:** 58188341

---

## Transcript

### Hook lifecycle

```
[User prompt arrives]  →  user_prompt_submit hook
[Claude's turn]
  → [Tool call starts]  →  pre_tool_use hook
  → [Tool result]       →  post_tool_use hook
  → [Tool fails]        →  post_tool_use_failure hook
[Claude's turn ends]   →  stop hook
```

Other available hooks: `permission_request`, `sub_agent_start`, `sub_agent_stop`.

---

### Example: blocking destructive Bash commands

```python
def check_bash(tool_input):
    if "rm -rf" in tool_input.get("command", ""):
        return {
            "hook_output": "Destructive command blocked.",
            "permission_decision": "denied",
            "reason": "rm -rf is a destructive command",
        }
    return  # empty return = no opinion, pass through

def log_tool(tool_name, tool_input):
    print(f"Tool used: {tool_name}")
    return  # no opinion

options = ClaudeAgentOptions(
    hooks={
        "pre_tool_use": [
            {"matcher": {"name": "Bash"}, "hooks": [check_bash]},
        ],
        "post_tool_use": [
            {"hooks": [log_tool]},  # no matcher = runs for ALL tools
        ],
    }
)
```

---

### Sub-agents

Sub-agents run with **isolated context windows** — the parent agent only sees the sub-agent's final result, not its intermediate tool calls.

**Spawning a sub-agent is called a "task"** (older API) or "agent" (current). Both are compatible — know both terms for the exam.

```python
from claude_agent_sdk import AgentDefinition

security_reviewer = AgentDefinition(
    description="Reviews code for security vulnerabilities. Call when code needs a security audit.",
    prompt="You are a security expert. Review the provided code and list vulnerabilities.",
    tools=["Read", "Grep"],
)

options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    agents={"security_reviewer": security_reviewer},
    hooks={
        "sub_agent_start": [{"hooks": [lambda: print("Sub-agent started")]}],
        "sub_agent_stop":  [{"hooks": [lambda: print("Sub-agent done")]}],
    }
)
```

**Key points:**
- Sub-agent starts with a fresh context.
- Tool calls inside the sub-agent are NOT reported to the parent.
- Only the final result propagates back.
- Use `AsyncIO` for hooks so sub-agent invocations don't block other processes.
