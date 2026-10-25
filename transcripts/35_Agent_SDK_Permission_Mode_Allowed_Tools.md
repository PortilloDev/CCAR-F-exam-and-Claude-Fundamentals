# Lecture 35: Agent SDK: Permission Mode & Allowed Tools

**Section:** Agentic Architecture  
**Lecture ID:** 58188357

---

## Transcript

### Permission gate flow

When a tool is about to be called, three gates are checked in order:

```
Gate 1: Is the tool in disallowed_tools?  → YES → BLOCKED (never called)
Gate 2: Is the tool in allowed_tools?     → YES → APPROVED (called without prompt)
Gate 3: What does permission_mode say?    → Depends on mode (see table)
```

---

### Permission modes

| Mode | Gate 3 behavior |
|------|----------------|
| **default** | Triggers `can_use_tool` hook — user can approve/deny at runtime |
| **acceptEdits** | Auto-approves file reads + edits; other tools still need approval |
| **plan** | Claude can generate plans, but cannot run other tools |
| **dontAsk** | Tools NOT in `allowed_tools` are auto-denied (no runtime prompt) |
| **bypassPermissions** | ALL tools approved — dangerous, use with extreme caution |
| **auto** | Classifier model decides automatically (same as Claude Code auto-approve) |

**Exam critical:** `dontAsk` + `allowed_tools` = only the listed tools run; everything else is silently denied. In `default` mode, unlisted tools still reach the `can_use_tool` hook and can be approved at runtime.

---

### Code example

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    tools=["Read", "Grep", "Bash"],        # what the agent CAN access
    allowed_tools=["Read", "Grep"],         # pre-approved (no prompt)
    disallowed_tools=["mcp.github.*"],      # always blocked
    permission_mode="dontAsk",              # everything not in allowed_tools → denied
)
```

---

### `can_use_tool` hook (Gate 3 — default mode only)

```python
def my_permission_hook(tool_name, tool_input):
    if tool_name == "Write" and "/etc/" in tool_input.get("path", ""):
        return {"permission_decision": "denied", "reason": "Writes to /etc are not allowed"}
    return {"permission_decision": "approved"}

options = ClaudeAgentOptions(
    hooks={"can_use_tool": [{"hooks": [my_permission_hook]}]},
    permission_mode="default",
)
```

---

### Subagent tool inheritance

- If `tools` is omitted or empty for a sub-agent → it **inherits all tools from the parent**.
- Always combine with `disallowed_tools` to narrow scope when inheriting.

---

### headless mode pattern (Claude Code CI/CD)

```python
# Typical headless pattern: pre-approve safe tools, deny everything else
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Grep", "WebFetch"],
    permission_mode="dontAsk",   # no interactive prompts in CI
)
```
