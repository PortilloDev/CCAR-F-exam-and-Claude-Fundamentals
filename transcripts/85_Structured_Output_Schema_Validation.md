# Lecture 85: Structured Output: Schema Validation

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217243

---

## Transcript

### Three error types in extraction

| Type | Example | Detectable by schema? |
|------|---------|----------------------|
| **Structural** | Missing field, wrong type (string in boolean), field misplaced | YES (partially) |
| **Misplaced** | Schema valid, but total placed inside a line item | NO |
| **Inconsistency** | Line items sum ≠ stated total | NO — semantic check required |

---

### Validation pipeline

```
Extract
  └── Semantic check (code logic)
        ├── PASS → continue
        └── FAIL → Is the missing value mandatory?
              ├── YES (no null allowed) → flag for human review
              └── NO → attempt repair
```

---

### Repair retry prompt structure

```
Original document:
<document>

Previous extraction:
<extraction>

Validation error:
"stated_total is 150 but line items sum to 145. Recheck and correct."

Re-extract and fix the discrepancy.
```

**Retry helps when:**
- Date in wrong format
- Value placed in wrong field
- Arithmetic error (single digit wrong)

**Retry does NOT help when:**
- Information is not in the document at all → send to human review

---

### Limits of schema validation

Schema cannot validate:
- Minimum/maximum numeric values
- Cross-field arithmetic (totals)
- Value placement correctness
- Invented values

These require semantic checks in application code or Pydantic validators.
