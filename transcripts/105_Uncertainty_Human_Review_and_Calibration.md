# Lecture 105: Uncertainty: Human Review & Calibration

**Section:** Context and Reliability  
**Lecture ID:** 58217293

---

## Transcript

### The hidden-subgroup problem

Aggregate accuracy of 97% looks safe — but:

| Document type | Volume | Accuracy |
|--------------|--------|---------|
| Standard invoices | 60% | 99% |
| Scanned receipts | 24% | 98% |
| Utility statements | 11% | 96% |
| **New vendor template** | **5%** | **60%** |

Aggregate score: 97%. New vendor template: 60%.  
**The aggregate hides the failing subgroup.**

---

### Stratified sampling (the fix)

Take ~3% of all documents, spread proportionally across every document type:

```
3% of standard invoices
3% of scanned receipts
3% of utility statements
3% of new vendor template   ← catches the 60% problem
```

**Benefits of stratified sampling:**
- Detects per-document-type failure rates.
- Catches novel patterns (new document types appearing in production).
- Finds cases where the auto-approval threshold is too permissive for a specific type.

---

### What confidence scores do and don't cover

| Confidence score covers | Confidence score does NOT cover |
|------------------------|-------------------------------|
| All required fields extracted | Field values that are incorrect |
| Schema structure valid | Arithmetic errors (line items ≠ total) |
| | Contradictory business-logic values |

For the latter two: use deterministic code checks (hooks) or send to human review.

---

### Anti-pattern (exam)

Self-rated confidence without labeled calibration data = unreliable. Always ground confidence scores in a labeled dataset.
