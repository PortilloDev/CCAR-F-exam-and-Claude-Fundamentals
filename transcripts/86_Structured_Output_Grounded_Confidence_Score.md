# Lecture 86: Structured Output: Grounded Confidence Score

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217245

---

## Transcript

### Per-field confidence (not document-level)

```json
{
  "vendor":        {"value": "Acme Corp",  "confidence": 0.95},
  "stated_total":  {"value": 150.00,       "confidence": 0.82},
  "purchase_order":{"value": null,         "confidence": 1.0}
}
```

**Why per-field:** document-level score hides which fields are uncertain.

---

### Calibration workflow

```
1. Let Claude produce a confidence score per field
2. Build a calibration dataset (labeled few-shot examples at different score levels)
3. Derive a threshold per field from that dataset
4. Compare incoming scores against threshold → route to automation or human review
```

---

### Anti-patterns (exam)

| Anti-pattern | Problem |
|-------------|---------|
| Pure self-rating (no calibration data) | No baseline to compare against |
| Single score for whole document | Too coarse to make field-level decisions |
| Threshold set by intuition | Not grounded in actual performance data |

---

### Routing decision matrix

| Source contradicts itself | Confidence | Action |
|--------------------------|------------|--------|
| No | Above threshold | Automate |
| Yes | High | Human review |
| Either | Below threshold | Human review |

---

### When to send to human review

- Field confidence score below threshold
- Field has a conflicting flag (e.g., null but still high confidence — something is wrong)
- Category is unclear
- Too many retries to repair

---

### Managing reviewer queue

When too many documents go to review (losing automation benefit):
1. **Prioritize** by business impact (how costly is the error?).
2. **Shrink queue** by adjusting the threshold slightly.
3. **Use reviewed labels** to fine-tune the model on edge cases.
