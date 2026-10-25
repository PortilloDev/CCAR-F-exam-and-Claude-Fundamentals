# Lecture 87: Structured Output: Learning From Dismissed Findings

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217177

---

## Transcript

### The feedback loop

```
Extraction / review pipeline
  └── Human reviewer dismisses finding
        └── Collect dismissed findings
              └── Detect pattern (add "detected_pattern" field to schema)
                    └── Build "do not flag" list + few-shot examples
                          └── Improve criteria in prompt
```

---

### Adding a detected_pattern field

```json
{
  "finding": "...",
  "detected_pattern": "try_except_flag"   // Claude categorizes the dismissal
}
```

As dismissed findings accumulate, group them by `detected_pattern` to find which criteria need improvement.

---

### Actions by dismissal rate

| Dismissal rate for a category | Action |
|-------------------------------|--------|
| High | Add to "do not flag" block; improve category criteria |
| Low | Category is working — high confidence |

---

### Evidence the improvement worked (generalization effect)

The prompt is ready when:
1. Excluded items are correctly excluded on cases **not in the few-shot set**.
2. Previously well-performing categories still work.
3. Dismissal rate drops over time.

**Not evidence:** only performs better on the exact examples provided (overfitting, not generalization).

---

### Exam pattern summary

```
False positives arriving → collect dismissed items
  └── Add detected_pattern field to schema
        └── Group by pattern
              └── High-dismissal pattern → add to "do not" block
                    └── Improve criteria + add few-shot boundary examples
                          └── Watch for generalization (new cases also excluded)
```
