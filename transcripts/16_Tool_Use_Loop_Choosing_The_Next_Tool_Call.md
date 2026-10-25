# Lecture 16: Tool-Use Loop: Choosing The Next Tool Call

**Section:** Tools and MCP  
**Duration:** ~6 min | **Lecture ID:** 58184791

---

## Transcript

For every turn when you call the messages API, you have three options when it comes to the decision to call any tool:

**Option A: `auto`** — Claude decides whether to call any tool or not.

**Option B: `any`** — Forces at least one tool call, but Claude chooses which one.

**Option C: `tool`** — Forces a specific named tool to be called.

**Decision tree:**
- Could Claude need a tool or not? → `auto` (let Claude decide freely)
- Must at least one tool be called but which doesn't matter? → `any`
- Must a specific exact tool be called? → `tool` (with name specified)

**Code example:**
```python
tool_choice={"type": "tool", "name": "extract_metadata"}  # force specific tool
tool_choice={"type": "any"}   # require at least one tool
tool_choice={"type": "auto"}  # Claude decides
```

**Parallel vs sequential tool calls:**
- If tool calls are **independent** of each other → run them in parallel (all in one turn, return all results in one user message with matching IDs)
- If tool calls are **dependent** on each other → run sequentially (separate API calls)

**Error handling in tools:**
```python
# Success case
{"is_error": False, "content": result}

# Error case  
{"is_error": True, "content": "Expected date format YYYY-MM-DD, got: '14 Sep'"}
```

**Error message quality:**
- ❌ BAD: "Operation failed" (too generic)
- ✓ GOOD: "Expected date format YYYY-MM-DD, received '14 Sep 2025'" (Claude can adapt)

**Tool robustness recommendations:**
- Communicate access errors clearly (wrong API key, misconfiguration)
- Empty result ≠ error (a search returning 0 results is still a successful call)
- Build retry logic for transient errors (timeouts) inside the tool itself
- Don't retry permission errors or business errors
- Distinguish access errors (can't reach the tool) from permission errors (reached but denied)

**Limits of `tool_choice`:**
- You cannot enforce the ORDER of tool calls (only which tools are called in a turn)
- For required sequences, enforce them programmatically in your code (separate turns)
- `tool_choice` is not available on Claude Fable 5.1 or Claude Methos 5.1 (the only exceptions)

**Diagnosing tool dispatch bugs:**
- Model requests a tool but result never returns → check that every tool result has a matching `tool_use_id`
- Check that you're appending ALL blocks from the assistant's response (not just the tool_use block)
- Tool results should be encoded as JSON with dictionaries and lists
