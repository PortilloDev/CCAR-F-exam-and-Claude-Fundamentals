# Lecture 37: Lab 1b: Build Agent + Hooks with Agent SDK + MCP Integration

**Section:** Agentic Architecture  
**Lecture ID:** 58217537

---

## Transcript

### Lab goal

Extend the Lab 1a MCP server to:
- Add a **wiki lookup tool** (4th tool).
- Build an agent using the **Agent SDK**.
- Assign built-in tools (`Grep`, `Glob`, `Read`) + MCP dev tools server.
- Configure `allowed_tools` and `dontAsk` permission mode.
- Add a **pre-tool hook** to record tool calls and optionally deny the wiki tool.

---

### Agent configuration

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    cwd=workspace_dir,
    max_turns=8,
    max_budget=0.10,          # 10 cents
    tools=["Grep", "Glob", "Read"],
    mcp_servers=[dev_tools_server],
    allowed_tools=[
        "mcp.dev_tools.*",    # all MCP tools pre-approved
        "Grep", "Glob", "Read",
    ],
    permission_mode="dontAsk",
)
```

---

### Lab-provided `labkit` helper

The course uses a `labkit` wrapper around the Agent SDK with two modes:
- `ask(question, tools, mcp_servers, allowed_tools)` — one-shot (`query()` internally).
- `converse(question, ...)` — multi-turn (`ClaudeSDKClient` internally, stores session on disk).

---

### Observed behavior (intentionally bad descriptions)

**Prompt:** "Has anyone reported a customer being charged twice?"

The agent called the MCP `lookup` tool with queries `"customer charged twice"` and a follow-up — and found the relevant ticket.

**Key lesson from Lab 1a:** With ambiguous tool descriptions, the model said "I don't have access to a ticketing system." With the Agent SDK and correct tool assignment, even vague descriptions work if the tools are explicitly listed.

**Lab 1b fix:** Rewrite tool descriptions using the 6-point checklist → model picks correct tools without being forced.

---

### Pre-tool hook pattern

```python
def record_and_gate_tool(tool_name, tool_input):
    print(f"[HOOK] Tool called: {tool_name}")
    if tool_name == "mcp.dev_tools.lookup_wiki":
        return {"permission_decision": "denied", "reason": "Wiki access disabled in this run"}
    return  # no opinion for all other tools
```

---

### Key takeaways

- Agent SDK wraps the loop, manages tool dispatch, and handles session state.
- `dontAsk` + `allowed_tools` = hard security boundary for production/headless runs.
- Hooks provide observability (logging) and control (blocking specific tool calls).
- Lab 1b proves that well-described MCP tools eliminate the need to force-specify tools in the prompt.
