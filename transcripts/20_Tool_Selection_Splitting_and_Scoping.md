# Lecture 20: Tool Selection: Splitting & Scoping Tools

**Section:** Tools and MCP  
**Duration:** ~6 min | **Lecture ID:** 58184803

---

## Transcript

**When to SPLIT a tool:**
- One tool doing too many jobs → Claude gets confused, wrong calls, messy input schema
- Rule: **one tool = one job**
- Example: `analyze_document` doing revenue figures, paragraph summaries, AND claim verification → split into 3 tools: `extract_data_points`, `get_summarized_content`, `get_claims_with_sources`

**When to CONSOLIDATE tools:**
- Multiple lookup tools with very similar input schemas → consolidate into one
- Pattern: different entity types but same lookup logic
- Example: `get_customer(id)` + `get_order(id)` → `lookup_entity(entity_type, id)`
- Consolidation works when the only differentiator is a type/record field

**Neither split nor consolidate when:**
- Tool already focuses on one specific job AND it's not a lookup task
- Consider improving the description first if issues arise

**Tool scoping principles:**
- Don't overload agents with tools — Anthropic's exam guide recommends 4-5 tools max per agent vs. 18+
- Remove ALL unused tools first
- Assign only tools appropriate for the agent's role

**Irreducible limitations:**
- Tool selection is always probabilistic (even with perfect descriptions, error rate is non-zero)
- Scripting a fixed tool order loses AI flexibility for edge cases
- Routing classifier helps but is not a complete fix
- For hard business rules (e.g., must verify customer before refund): encode as input validation inside the tool itself — deterministic gatekeeping at the tool level

**Diagnosing dispatch bugs:**
- Tool was triggered in logs but Claude keeps asking for the same data → result likely never made it back
- Check: every `tool_use` request has a matching `tool_result` with the correct ID
- Check: ALL blocks from the assistant message were appended to history (not just `tool_use`)
- Tool results must be JSON-encoded with dictionaries and lists
