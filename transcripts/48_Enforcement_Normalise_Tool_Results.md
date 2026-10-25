# Lecture 48: Enforcement & Handoff: Normalise Tool Results

**Section:** Agentic Architecture  
**Lecture ID:** 58217015

---

## Transcript

### The problem

Multiple tools return the same type of data in different formats:

| Tool | Timestamp format | Status format |
|------|-----------------|--------------|
| `get_customer` | Unix timestamp | — |
| `lookup_order` | ISO 8601 | — |
| `check_payment` | — | Numeric code (200, 404, 500) |

If Claude receives raw Unix timestamps or numeric status codes, it must compute conversions itself — extra tokens, extra reasoning, extra error surface.

---

### Solution: `post_tool_use` normalization hook

```python
STATUS_MAP = {200: "OK", 404: "Not found", 500: "Error"}

async def normalize_tool_output(input_data, tool_use_id, context):
    tool_response = input_data["tool_response"]
    
    # Normalize timestamp: Unix → ISO 8601
    raw_ts = tool_response.get("created")
    if isinstance(raw_ts, (int, float)):
        iso_ts = datetime.utcfromtimestamp(raw_ts).isoformat() + "Z"
    else:
        iso_ts = raw_ts  # already ISO
    
    # Normalize status: numeric → natural language
    status_code = tool_response.get("status")
    readable_status = STATUS_MAP.get(status_code, str(status_code))
    
    out = {**tool_response, "created": iso_ts, "status": readable_status}
    
    return {
        "hookSpecificOutput": {
            "hookEventName": "PostToolUse",
            "updatedToolOutput": out,    # replaces the original tool result
        }
    }
    # return {} → no change to tool result
```

---

### When to use `post_tool_use` for normalization

✅ Normalize timestamps, status codes, units across different tools.  
✅ Enrich responses with additional computed fields.  
✅ Remove fields Claude doesn't need (reduces tokens).  
❌ Cannot stop or undo an already-executed tool call.  
❌ Cannot block a refund that already happened — use `pre_tool_use` for that.

---

### Why ISO 8601 + natural language status?

- LLMs process natural language more reliably than numeric codes.
- Avoids model needing to run code to convert Unix timestamps.
- Fewer tokens, fewer reasoning steps, lower hallucination risk.
