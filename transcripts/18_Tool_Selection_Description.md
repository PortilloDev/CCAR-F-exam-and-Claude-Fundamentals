# Lecture 18: Tool Selection: Description

**Section:** Tools and MCP  
**Duration:** ~5 min | **Lecture ID:** 58184795

---

## Transcript

The best way to improve how Claude selects the right tools for the right task is to have the right description in place.

**Start with the name:**
- Use a clear name with no ambiguity
- Each name should isolate its domain — no similar names between tools
- BAD: `DocSearch` (internal) vs `GeneralSearch` (web) — too similar
- GOOD: `SearchKnowledgeBase` vs `WebSearch` — clearly distinct

**The 6-Point Checklist for Tool Descriptions:**

1. **What (Job)** — What does the tool do? E.g., "Searches through the internal knowledge base."
2. **When (Condition)** — When should it be triggered? E.g., "Use when looking for policy documents or company procedures."
3. **When NOT (Inverse condition)** — E.g., "Do not use for general knowledge questions."
4. **Inputs** — Best practices for calling the parameters (these belong in the description, not the schema).
5. **Outputs** — Describe the output format (e.g., "Returns matching documents with titles, snippets and relevance scores").
6. **Edge cases** — Known cases from experience and evaluation.

**Example — bad description:**
> "Search function"

**Example — good description:**
> "Search the company knowledge base for internal documentation, policies and procedures. Use this when the user asks about company-specific processes. Do not use for general knowledge questions. Returns matching documents with titles, snippets and relevance scores."

**Always add a boundary sentence** to your description — it clarifies what the tool should NOT be used for and reduces overlap with neighboring tools.

**Boundary sentence examples:**
- `search_knowledge_base` → "Do not use for general knowledge questions."
- `get_current_datetime` → "Do not use to calculate future dates by adding a time frame — that is handled by a separate tool."
- `add_reminder` → "Do not use to override existing reminders."

**Key takeaway:** Good descriptions = better tool selection accuracy = fewer tokens wasted on wrong calls.
