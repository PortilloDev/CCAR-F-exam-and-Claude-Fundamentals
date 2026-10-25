# Lecture 90: Scale & Review Architecture: Message Batch API

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217249

---

## Transcript

### What the Batch API provides

- Send many requests in one block — up to 50% cost savings.
- Each request gets a **custom ID** (you define it — use the document ID).
- Results come back in a **different order** than submitted → always use custom ID to map results.
- SLA: results delivered within **24 hours**.

---

### Code pattern

```python
# Create batch
batch = client.messages.batches.create(requests=[
    {
        "custom_id": "doc-001",
        "params": {
            "model": "claude-opus-5",
            "max_tokens": 4096,
            "messages": [{"role": "user", "content": document_1}],
            "tools": [...],
            "tool_choice": {"type": "tool", "name": "extract_invoice"}
        }
    },
    # ... more requests
])

# Poll for completion
status = client.messages.batches.retrieve(batch.id)

# Retrieve results
for result in client.messages.batches.results(batch.id):
    custom_id = result.custom_id      # maps back to your document
    content   = result.result.message.content
```

---

### Limitations

| Limitation | Detail |
|-----------|--------|
| No multi-turn | One single model response per request |
| One forced tool call allowed | For extraction — but no further tool loop |
| 24h SLA | Can finish faster; can have item failures requiring resubmission |
| Order not guaranteed | Always use custom ID |
| Cannot stream results | Poll for completion |

---

### When NOT to use Batch API

- You need the tool result to continue (multi-turn).
- Results must be handed immediately to a reviewer.
- You don't know how many turns the agent needs.
- Deadline is less than ~30 hours away (24h batch + downstream processing).
