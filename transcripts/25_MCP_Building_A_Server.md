# Lecture 25: MCP: Building A Server

**Section:** Tools and MCP  
**Lecture ID:** 58188305

---

## Transcript

### Should you build a custom MCP server?

**Decision tree:**
1. Does a standard MCP server already exist for this service? → **Use it.**
2. Is there an open-source MCP server on GitHub? → **Evaluate it** (check community trust; beware of malware in unvetted repos).
3. Does the existing server's schema/design fit your requirements? → If yes, **use it**. If no, **build your own.**
4. Is this an internal API with no public MCP server? → **Build your own.**

**Build your own when:**
- Internal API not yet exposed for MCP.
- Specific business logic not yet in code.
- You need to customize schema (names, arguments, outputs).

---

### Building with FastMCP

```python
from fastmcp import FastMCP, Field

mcp = FastMCP("MyServerName")

@mcp.tool()
def ReadDocContents(
    doc_id: str = Field(description="The unique identifier of the document to read")
) -> str:
    """Reads the contents of an internal document by its ID.
    Use when the user requests a specific document. Returns the full text."""
    try:
        # ... implementation ...
        return content
    except Exception as e:
        raise ValueError(f"Could not read document '{doc_id}': {e}")
```

**FastMCP auto-generates** the tool schema (name, description, input schema) from the decorator and `Field` descriptions — no manual schema writing required.

---

### Testing with MCP Inspector

**Workflow:**
1. Decorate your functions with `@mcp.tool()`.
2. Start your local MCP server.
3. Run MCP Inspector via CLI → it listens on a proxy port + a frontend port.
4. Open the Inspector UI in the browser → connect to your server.
5. Verify:
   - All tools are listed.
   - Tools execute correctly.
   - Resources catalogue is accessible (correct MIME type, usually JSON).
   - Prompts are accessible (if used).
6. Connect an AI client (Claude Code or Claude Desktop) to the local server.
7. Check the history pane for tool calls.
8. Iterate until everything works, then deploy the server remotely.
