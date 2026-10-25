# Lecture 23: MCP: What MCP Solves

**Section:** Tools and MCP  
**Lecture ID:** 58188295

---

## Transcript

MCP stands for **Model Context Protocol**. The goal: make it easy to integrate tools from other providers without rewriting schemas and glue code for every application.

**The metaphor:** MCP is a USB-C adapter for agentic tools.

---

### The N×M problem (without MCP)

- N applications × M services = N×M individual tool integrations.
- Each integration requires understanding the API schema and writing the function.
- Example: integrating GitHub alone requires ~8 schemas + functions.

**With MCP:** Applications connect to an MCP server; the server handles the schema. The application only needs to know the MCP server URL.

---

### Architecture components

| Component | Role |
|-----------|------|
| **Host** | The main application (Claude Desktop, your app). Contains an MCP client. |
| **Client** | Relay inside the host that communicates with the MCP server. |
| **MCP Server** | Exposes tool names and schemas; connects to the real service (e.g., GitHub). |
| **LLM (Claude)** | Provides intelligence; decides which tools to call via the Messages API. |

**End-to-end flow:**
1. Host connects to MCP server → gets list of tool names + schemas.
2. Host calls Claude Messages API with user question + tool schemas.
3. Claude decides to call a tool → host client relays to MCP server.
4. MCP server calls the real API (e.g., GitHub) → returns response.
5. Response relayed back as a `tool_result` → Claude continues.

---

### Transport modes

| Mode | When to use |
|------|-------------|
| **stdio** | Local MCP server (fast, but consumes local resources) |
| **HTTP / Streamable HTTP** | Remote MCP server (HTTP + SSE for streaming responses) |

**SSE (Server-Sent Events):** Unidirectional streaming of MCP results back to the agent — faster streaming experience.

---

### MCP vs. plain tool use

MCP is complementary, not a replacement:
- Classic tool use still works for small code snippets exposed directly.
- MCP is the standardization layer for larger integrations and reusable servers.
- Most major SaaS apps (Asana, Salesforce, Jira, GitHub, Slack) now have MCP servers.

**Implication for your app:** Tool definitions and glue code move into the MCP server. Your application only needs an MCP client.
