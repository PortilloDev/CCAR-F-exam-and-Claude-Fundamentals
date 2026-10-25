# Study Doc — Section 2: Tools and MCP

**Domain:** Tool Design and MCP Integration  
**Files:** 12–28

---

## The Tool-Use Loop

### Four components

| Component | Who builds it |
|-----------|--------------|
| Tool function (the logic) | You |
| Tool schema (name, description, input_schema) | You |
| Dispatcher (routes tool calls to real functions) | You (or Agent SDK / Claude Code) |
| Loop (orchestrates turns) | You (or Agent SDK / Claude Code) |

### Flow per turn

```
1. Send messages + tools to Claude
2. stop_reason == "tool_use" → extract id, name, input
3. Dispatch to real function
4. Send back tool_result with same tool_use_id (role: "user")
5. Repeat until stop_reason == "end_turn" or "max_tokens"
```

**EXAM TRAP:** Tool results are sent with role `user` — there is no `tool` role.  
**EXAM TRAP:** Always include `tool_use_id` in the result — Claude matches by ID.  
**EXAM TRAP:** Resend full tool schemas on every call (stateless API).  
**EXAM TRAP:** Always store the full assistant message (including tool_use blocks) in history.

---

## Tool Definition — 3 Parts

```python
{
    "name": "search_knowledge_base",
    "description": "...",   # WHEN and how to call
    "input_schema": {       # HOW: parameters Claude fills
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "The search term"}
        },
        "required": ["query"]
    }
}
```

---

## Tool Description — 6-Point Checklist

1. **What** — what does the tool do?
2. **When** — when should it be triggered?
3. **When NOT** — inverse condition (boundary sentence)
4. **Inputs** — best practices for parameters
5. **Outputs** — format of the return value
6. **Edge cases** — known tricky cases

**EXAM TRAP:** Vague descriptions ("Search function") → wrong tool selection.  
**EXAM TRAP:** Never force tool_choice to fix wrong selection — fix the description.

---

## Diagnosing Wrong Tool Selection

```
1. Check system prompt — any keyword over-triggering a tool?
2. Check for overlapping tool names/descriptions → rename + rewrite
3. Expand descriptions (add all 6 checklist points)
4. Split into dedicated agents with scoped tools
5. Few-shot examples (last resort — high cost, high latency)
```

---

## Split vs. Consolidate vs. Scope

| Decision | When |
|----------|------|
| **Split** | Tool does too many jobs (one tool = one job) |
| **Consolidate** | Multiple similar lookup tools with same input pattern |
| **Scope** | Remove unused tools; 4-5 tools max per agent |

**EXAM TRAP:** `analyze_document` doing 3 different jobs → split into 3 tools.  
**EXAM TRAP:** `get_customer(id)` + `get_order(id)` → consolidate into `lookup_entity(type, id)`.

---

## Built-In Tools (Claude Code + Agent SDK)

| Tool | Does |
|------|------|
| **Glob** | Finds files by path/name pattern |
| **Grep** | Searches content inside files |
| **Read** | Reads full file contents |
| **Edit** | Replaces a unique anchor in a file |
| **Write** | Replaces entire file (when Edit anchor not unique) |
| **Bash** | Runs shell commands |

**Pattern:** Glob first (find files) → Grep (find content) → Read only if needed.  
**EXAM TRAP:** Grep = content inside files. Glob = file names/paths.  
**EXAM TRAP:** Empty result ≠ failure. Never return an error for zero results.

---

## MCP — Model Context Protocol

### Problem it solves

Without MCP: N apps × M services = N×M integrations.  
With MCP: Each service publishes one MCP server → any app connects via standard URL.

### Architecture

```
Host (Claude Desktop / your app)
  └── MCP Client
        └── MCP Server (exposes tools, resources, prompts)
              └── Real service (GitHub, Jira, etc.)
```

### Three primitives

| Primitive | Controlled by | Use case |
|-----------|--------------|---------|
| **Tools** | The AI model | Model decides when to call |
| **Resources** | The application | App pre-selects documents before the request |
| **Prompts** | The user | Slash commands that trigger packaged workflows |

**Resources rule:** use resources when the app can pre-select content. Use tools when the model must dynamically fetch.

### Transport modes

| Mode | Use when |
|------|---------|
| `stdio` | Local MCP server |
| `HTTP / Streamable HTTP` | Remote MCP server |

---

## MCP Error Types (EXAM CRITICAL)

| Type | Cause | Retriable | Model action |
|------|-------|-----------|-------------|
| **Transient** | Timeout, rate limit | YES (backoff) | Retry with increasing wait |
| **Validation** | Bad input format | YES (fix input) | Correct arguments, retry |
| **Business** | Policy limit hit | NO | Explain to user, offer alternative |
| **Permission** | Agent lacks access | NO | Escalate to human |

**EXAM TRAP:** Empty result ≠ error. `isError` must be `false` for zero results.  
**EXAM TRAP:** Treating failure as empty → model misses real errors (loops or gives wrong info).

```json
// Correct error format
{
  "error_category": "business",
  "retriable": false,
  "message": "Refund window expired: 45 days ago, policy is 30 days",
  "alternative": "Offer store credit or escalate"
}
```

---

## MCP Configuration

### Project-scoped (`.mcp.json`) — share with team

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {"GITHUB_TOKEN": "$GITHUB_TOKEN"}
    }
  }
}
```

**Always use env vars for credentials. Never hardcode.**

### Permission patterns

```
mcp.github.create_issue   → one tool
mcp.github.*              → all tools on server
mcp.github.get_*          → all tools starting with get_
```

**EXAM TRAP:** All MCP tools load at session start — trim unused ones to reduce confusion and token waste.
