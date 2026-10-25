# Lecture 27: MCP: Integrating Servers

**Section:** Tools and MCP  
**Lecture ID:** 58188315

---

## Transcript

### Project-scoped configuration (`.mcp.json`)

Create a `.mcp.json` file per project. It can be shared with the team.

**Always use environment variables for credentials** — never hardcode them in the file:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "my-remote-server": {
      "url": "https://my-mcp-server.example.com"
    }
  }
}
```

- **Local server:** use `command` + `args` (e.g., `npx`).
- **Remote server:** use `url` instead of `command`.

---

### Global configuration (`~/.claude/claude.json`)

Stores MCP servers that load for **every project**. Project-scoped servers are additive on top.

**Example:** `node-mcp-server` always on globally; `github` only for one specific project.

---

### Three ways to register an MCP server

1. **Claude Code CLI:**
   ```bash
   claude mcp add --scope user my-server npx @scope/my-server
   ```
   `--scope user` → saves to global `~/.claude/claude.json`.

2. **JSON config file:** Edit `.mcp.json` or `claude.json` directly.

3. **Agent SDK:** Define MCP servers inside your agent definition (same fields as JSON config).

---

### Permission controls (wildcards)

Pattern: `mcp.<server-name>.<tool-name>`

| Pattern | Scope |
|---------|-------|
| `mcp.github.create_issue` | One specific tool |
| `mcp.github.*` | All tools on the server |
| `mcp.github.get_*` | All tools starting with `get_` |

**Best practices:**
- Fewer allowed tools = fewer errors and less confusion.
- All tools from all connected MCP servers are loaded at session start — trim unused ones.
- All tool descriptions should follow the 6-point checklist (same as regular tools). Vague descriptions (e.g., "search tool") can overlap with built-in tools like Grep.
