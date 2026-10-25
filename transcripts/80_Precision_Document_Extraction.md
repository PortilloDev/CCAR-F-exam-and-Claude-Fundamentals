# Lecture 80: Precision: Document Extraction

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217169

---

## Transcript

### Two failure modes in document extraction

| Failure | Cause | Risk |
|---------|-------|------|
| Claude **invents** a value | Field is required but absent from document; retry pressure increases hallucination likelihood | High — silent error |
| Claude returns **empty field** but value was present | Layout not understood | Field missed |

**Exam pitfall:** recommending a retry when a field is absent from the document — retry increases hallucination risk, not accuracy.

---

### Few-shot examples for different layouts

When extracting citations, one example per layout type:

| Layout | Description | Example fields |
|--------|-------------|----------------|
| Inline | Statement and source in same sentence | value, source, type="inline" |
| Bibliography | Enumerated reference at end of document | value, source="reference 1", type="bibliography" |
| Embedded | Document itself is the source | value, source=document title, type="embedded" |

**Pattern:** one few-shot prompt per document type. Writing the examples is straightforward — identifying which layouts exist is the hard part.

---

### Informal measurements example

```json
// Input: "two handfuls of rice"
{
  "amount": 100,
  "unit": "grams",
  "original_text": "two handfuls of rice",
  "precision": "approximation"
}

// Input: "a pinch of salt"
{
  "amount": 0.4,
  "unit": "grams",
  "original_text": "a pinch of salt",
  "precision": "approximation"
}
```

---

### Decision tree for missing fields (exam)

```
Is the value absent in the extracted output?
└── Was it in the original document?
    ├── YES (value was there, extraction failed)
    │   └── Layout issue → add few-shot prompts for that layout
    └── NO (value was never in the document)
        ├── Option A: Preserve the absence (return null) — extraction worked correctly
        └── Option B: Add an additional data source to find the value
```

**Never:** retry harder or improve instructions when the value isn't in the document — the extraction succeeded; the document is the problem.
