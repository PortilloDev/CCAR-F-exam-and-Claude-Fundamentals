# Lecture 88: Lab 5: Document Extraction Pipeline with Messages API

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217531

---

## Transcript

### Lab objective

Build an intelligent document extraction pipeline using the Messages API directly. Input: scanned receipts (unstructured text). Output: structured JSON.

**Requires:** Anthropic API key (not OAuth token) — add to `.env` file.

---

### Step 1: Naive prompt approach (unreliable)

```python
response = client.messages.create(
    model="claude-opus-5",
    messages=[{"role": "user", "content": f"Extract this invoice as JSON:\n{document}"}]
)
```

**Problem:** response is prose — JSON mixed with explanatory text. Unparseable reliably.

---

### Step 2: Forced tool call (exam-preferred approach)

```python
schema = {
    "type": "object",
    "properties": {
        "vendor":        {"type": "string"},
        "invoice_number":{"type": "string"},
        "stated_total":  {"type": "number"}
    },
    "required": ["vendor", "invoice_number", "stated_total"]
}

response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    messages=[{"role": "user", "content": f"{document}\n\nExtract the record."}],
    tools=[{"name": "extract_invoice", "description": "Extract invoice", "input_schema": schema}],
    tool_choice={"type": "tool", "name": "extract_invoice"}
)
extracted = response.content[0].input   # Structured output — no prose
```

---

### Step 3: Pydantic validation (alternative)

```python
class LineItem(BaseModel):
    name: str
    price: float

class Invoice(BaseModel):
    vendor: str
    stated_total: float
    line_items: list[LineItem]

# Use model_json_schema() to generate the tool input_schema
schema = Invoice.model_json_schema()
# ... same forced tool call ...
# Validate with:
Invoice.model_validate(tool_block.input)
```

---

### Handling missing fields

**Required string field absent → hallucination:**
```json
{"purchase_order": "unknown"}  // invented
```

**Fix — allow null:**
```json
"purchase_order": {"type": ["string", "null"]}
```
Result: `{"purchase_order": null}` — correct, honest.

---

### Enum with missing categories

```json
"reason": {
    "type": "string",
    "enum": ["damage", "wrong_item", "overcharge", "short_delivery", "unclear", "other"]
}
```

When no existing value fits: returns `"other"`.  
When ambiguous: returns `"unclear"`.  
Both prevent forcing an inappropriate category.

---

### Key takeaways

1. Plain prompt → prose JSON = unreliable.
2. Forced tool call → `tool_block.input` = reliable structured output.
3. Nullable types and enum escape values prevent hallucination of missing data.
4. Clean up API key after the lab (revert to OAuth for other labs).
