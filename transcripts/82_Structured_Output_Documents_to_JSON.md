# Lecture 82: Structured Output: Documents to JSON

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217221

---

## Transcript

### Messages API — JSON schema approach

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,       # Too small → output truncated
    messages=[{"role": "user", "content": document_text}],
    output_config={"type": "json_schema", "schema": invoice_schema}
)
# Parse the text block as JSON
result = json.loads(response.content[0].text)
```

---

### Messages API — Pydantic approach

```python
from pydantic import BaseModel

class LineItem(BaseModel):
    name: str
    price: float

class Invoice(BaseModel):
    vendor: str
    stated_total: float
    line_items: list[LineItem]

# parse() method uses Pydantic models — no manual schema writing
response = client.messages.parse(
    model="claude-opus-5",
    response_model=Invoice,
    messages=[...]
)
result = response.parsed_output   # typed Invoice object
```

---

### Semantic validation (critical — schema does NOT check this)

```python
# Schema passes even if totals are wrong
extracted_total = invoice.stated_total
computed_total = sum(item.price for item in invoice.line_items)
if abs(extracted_total - computed_total) > 0.01:
    raise ValueError(f"Total mismatch: stated={extracted_total}, computed={computed_total}")
```

**Key point:** structured output guarantees field types and presence, not factual correctness.

---

### Agent SDK — structured output

```python
options = ClaudeAgentOptions(
    output_format=Invoice.model_json_schema()
)
result = await agent.query(prompt, options)
structured = result.structured_output   # typed output
```

**Retry difference:**
| | Messages API | Agent SDK |
|--|-------------|-----------|
| Retry on extraction failure | Manual | Automatic (`max_structured_output_retries`) |
| Access structured output | Parse `content[0].text` as JSON | `result.structured_output` |

---

### max_tokens warning

If `max_tokens` is too small, the structured output is truncated and JSON parsing fails. Size `max_tokens` generously for extraction tasks.
