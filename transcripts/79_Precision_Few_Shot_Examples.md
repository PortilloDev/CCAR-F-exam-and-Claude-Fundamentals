# Lecture 79: Precision: Few-Shot Examples

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217157

---

## Transcript

### When few-shot examples help

Few-shot examples resolve ambiguous boundary cases — they leave nothing to interpret. Use them only for cases that are still vague after criteria improvements. Things that already work well don't need examples.

**Decision flow:**

```
Are tool descriptions well-defined?
└── NO → Fix tool descriptions first
YES → Does the instruction have categorical criteria?
      └── NO → Add criteria first
      YES → Are there still false positives at a specific boundary?
            └── YES → Add few-shot examples for that boundary only
```

---

### Structure of a few-shot example

```python
# Example: code review finding with suggested fix
{
  "line": 42,
  "issue": "Loop over payment list uses O(n²) lookup",
  "severity": "high",
  "suggested_fix": "Replace inner loop with a dict lookup keyed by payment_id"
}
```

Add XML tags to highlight examples in the prompt:

```xml
<example>
  <input>...</input>
  <output>...</output>
</example>
```

---

### Boundary case example (JavaScript)

```javascript
// FINE — idiomatic truthiness
if (x.active) { ... }

// FLAG — strict boolean type check (non-idiomatic)
if (x.active === true) { ... }

// FINE — strict equality with ===
if (x.active === true) { ... }   // depends on context
```

One example covering this nuance is enough — Claude generalizes the pattern.

---

### Customer support domain example

| Input | Recommended action | Why |
|-------|-------------------|-----|
| "My order is broken" | Escalate to damage claim flow | "Broken" = physically damaged item in this domain |
| "Get me a manager" | Hand off to human agent | Signals escalation request, not a solvable ticket |

---

### Generalization effect (exam key point)

Two or three boundary examples → Claude generalizes to similar cases it hasn't seen. Evidence the prompt is working:
- Fewer false positives on cases NOT in the few-shot set.
- Previously well-performing categories still perform well.
- Excluded items are correctly excluded.

**Not evidence:** tone improved, or it only performs better on the exact examples provided (no generalization).
