# Lecture 28: Lab 1a: Build MCP Server and Connect to Claude Code

**Section:** Tools and MCP  
**Lecture ID:** 58217515

---

## Transcript

### Lab structure

A fictional company workspace containing:
- **Ticket data** (JSON)
- **Architecture Decision Records** (ADR markdown files)
- **Service catalog** (JSON with service owners and escalation contacts)

Three tools are exposed via an MCP server:
- `lookup` — searches tickets
- `service_catalog` — looks up service responsibilities
- `architecture_decisions` — reads ADR documents

A custom `tool_error` helper structures error responses with category, retriability, message, and next-step guidance.

---

### Error category helper pattern

```python
RETRIABLE = {
    "transient": True,
    "validation": True,
    "business": False,
    "permission": False,
}

NEXT_ACTIONS = {
    "validation": "Call again with one of the listed names.",
    "permission": "Ask a payments engineer or escalate.",
    "business": "Escalate to a manager.",
    "transient": "Retry with backoff.",
}

def tool_error(category, message):
    return {
        "error_category": category,
        "retriable": RETRIABLE[category],
        "message": message,
        "next_step": NEXT_ACTIONS[category],
    }
```

---

### Validation error example

```python
# service_catalog tool — input validation
if service_name not in KNOWN_SERVICES:
    raise ValueError(tool_error(
        "validation",
        f"'{service_name}' not found. Valid names: {', '.join(KNOWN_SERVICES)}"
    ))
```

---

### Intentionally ambiguous descriptions (Lab 1a)

The lab intentionally uses bad tool descriptions (`lookup things`, `lookup decisions`) to demonstrate tool confusion. The model defaults to generic web search instead of the correct tool.

**Observed behavior:** With ambiguous descriptions, the model says "I don't have access to a ticketing system." When forced (`Use the dev_tools MCP tool for lookup`), it finds the ticket correctly.

**Fix (Lab 1b):** Rewrite descriptions using the 6-point checklist.

---

### Connecting to Claude Code

**Project scope:**
```bash
# .mcp.json is auto-detected in the project root
claude   # → slash mcp → shows "project mcp" section with dev_tools server
```

**User scope (personal, all projects):**
```bash
claude mcp add --scope user dev-tools-personal npx ./servers/dev_tools_server.py
# Adds to ~/.claude/claude.json
# Remove with:
claude mcp remove dev-tools-personal
```

**Verifying in Claude Code:**
- `/mcp` → shows project MCP servers and user MCP servers
- Click a server → "View tools" → confirms the 3 tools are registered

---

### Key takeaways

- MCP servers can be scoped per project (`.mcp.json`) or globally (`~/.claude/claude.json`).
- Well-defined tool descriptions are essential — the lab proves that vague names cause the model to ignore correct tools entirely.
- The `tool_error` helper pattern makes error responses structured, retriable, and actionable.
- Lab 1b will fix the descriptions and demonstrate the improvement.
