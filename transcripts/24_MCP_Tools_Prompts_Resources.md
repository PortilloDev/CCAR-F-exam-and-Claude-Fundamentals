# Lecture 24: MCP: Tools, Prompts and Resources

**Section:** Tools and MCP  
**Lecture ID:** 58188303

---

## Transcript

MCP has three **primitives**:

| Primitive | Controlled by | Description |
|-----------|--------------|-------------|
| **Tools** | The AI model | Model decides which tool to call |
| **Resources** | The application | App selects documents to attach to the session (like a drop-down menu) |
| **Prompts** | The user | User triggers a packaged workflow via slash command |

---

### Resources in detail

Resources are documents exposed by the MCP server. They are **application-controlled**: the host application selects them before the request reaches the model.

**URI formats for resources:**
- Single document: `documents/doc_id` (parameterized)
- Folder listing: `documents/` (all documents in a folder)
- MIME types are preserved in transit: HTML, PDF, JSON, etc.

**The catalogue advantage:**
- Without catalogue: model must ask which files exist → read document → answer. (Extra step.)
- With catalogue: application selects the document before sending the request → model answers directly. (One step saved.)

**Good use cases for resources:**
- Ticketing systems: index of issues with summaries, IDs, status, owner.
- Documentation: hierarchy of the documentation structure for faster navigation.
- Databases: schema hidden behind a resource so the agent can work with tables directly.

**Rule:** Use resources when the application can pre-select the content. Use tools when the model must dynamically choose what to fetch.

---

### Prompts in detail

Prompts are packaged workflows stored in the MCP server. The user activates them via a slash command. They can contain full multi-step procedures, making complex workflows reusable and triggerable without re-describing them each time.
