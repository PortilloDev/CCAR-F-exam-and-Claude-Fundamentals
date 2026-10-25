# Lecture 83: Structured Output: Extraction as a Tool

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217235

---

## Transcript

### Why "extraction as a tool call" is the exam-preferred pattern

Asking in a plain prompt for JSON produces **prose** — text before and after the JSON block. This is not reliably parseable. The correct approach: simulate a tool call.

---

### The pattern

```python
# 1. Define the extraction tool with exact output schema
tools = [{
    "name": "extract_invoice",
    "description": "Extract structured invoice data",
    "input_schema": {
        "type": "object",
        "properties": {
            "vendor":       {"type": "string"},
            "total":        {"type": "number"},
        },
        "required": ["vendor", "total"],
        "additionalProperties": False,
        "strict": True
    }
}]

# 2. Force the tool call
response = client.messages.create(
    model="claude-opus-5",
    messages=[{"role": "user", "content": document_text + "\n\nExtract the record."}],
    tools=tools,
    tool_choice={"type": "tool", "name": "extract_invoice"}
)

# 3. Read the tool input — this IS the structured output
tool_block = response.content[0]   # tool_use block
extracted = tool_block.input       # {"vendor": "...", "total": 150.0}
# No need to call the tool — the conversation ends here
```

---

### Key parameters

| Parameter | Purpose |
|-----------|---------|
| `additionalProperties: false` | Prevents extra unexpected fields |
| `strict: true` | Stops the flow if any required field is missing |
| `tool_choice: {type: "tool", name: "..."}` | Forces exactly this tool to be called |

---

### Semantic checks still needed

The schema enforces structure, not correctness. Check separately:
- Arithmetic (line items sum ≠ stated total)
- Values in wrong fields
- Invented values (use few-shot prompts at the boundary)

---

### Multi-tool choice strategy

| Situation | `tool_choice` |
|-----------|-------------|
| Document type unknown | `"auto"` — Claude picks one of [invoice, contract, receipt] |
| Document type known in advance | Force exactly one tool |

**Note (exam):** newer models may not support forced tool choice — use `response_format` or `auto` mode instead.
