# Lecture 84: Structured Output: Missing Fields

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217241

---

## Transcript

### The hallucination trap

If a field is `required` with type `string` and the value is absent from the document, Claude **invents** a value. Prevention: allow null or add enum escape values.

---

### Four strategies for missing data

| Strategy | Schema | Use when |
|----------|--------|---------|
| **null** | `"type": ["string", "null"]` | Value simply not in document |
| **unclear** | enum value `"unclear"` | Ambiguous — cannot classify |
| **other** | enum value `"other"` | No existing category fits |
| **omit** | Remove `required` dependency | Field is truly optional |

---

### Nullable field pattern

```json
{
  "properties": {
    "purchase_order": {"type": ["string", "null"]},
    "issued_on":      {"type": ["string", "null"]}
  },
  "required": ["purchase_order", "issued_on"]
}
```

If the document lacks the field → Claude returns `null` instead of inventing a value.

---

### Enum with escape values

```json
{
  "reason": {
    "type": "string",
    "enum": ["damage", "wrong_item", "overcharge", "short_delivery", "unclear", "other"],
    "description": "Use 'unclear' when ambiguous and cannot classify. Use 'other' when none of the existing values fit."
  }
}
```

**Why description matters:** without it, Claude may not know when to use `unclear` vs. `other`.

---

### Prompt-level normalization

Use the prompt to standardize formats for string fields:

| Field type | Normalization instruction |
|-----------|--------------------------|
| Dates | "Return absolute date in ISO 8601 format (YYYY-MM-DD)" |
| Money | "Return numeric value and currency separately" |
| Percentages | "Return as decimal fraction (0.15 for 15%)" |
| Quantities | "Return in metric units" |

Schema cannot enforce these transformations — the prompt does.
