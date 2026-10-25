# Lecture 94: Scale & Review Architecture: Test Small & Batch Corpus

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217267

---

## Transcript

### The two-step process

1. **Optimize prompt on a small representative sample** (synchronous API — fast feedback).
2. **Run at scale on the full corpus** (Batch API — cost-efficient).

Never run the full corpus first — a prompt error on thousands of documents is extremely expensive to fix.

---

### What to include in the sample

| Sample type | Why |
|-------------|-----|
| One example of each document type | Covers all layouts |
| Longest documents | Finds context window overflow early |
| Different layout options (scan, multi-column, header/footer) | Finds layout-specific extraction errors |

Use the **synchronous API** for the sample — fast turnaround, not worth waiting 24h for Batch.

---

### Iterate on the sample

```
Run prompt on sample → review results → fix prompt/schema → repeat
```
Until one clean pass. Then submit full corpus to Batch API.

---

### Handling batch results

Sort results by status:

| Status | Action |
|--------|--------|
| Succeeded | Still validate content (semantic checks) |
| Invalid request error | Document too large (context overflow) — chunk, resubmit |
| Other technical error | Resubmit as-is (prompt was fine) |
| Expired (24h timeout) | Resubmit; consider high-priority items as synchronous |

---

### Resubmission rules

- **Track custom IDs** of all failed/expired/invalid items.
- Resubmit only failed items — never the full job.
- For invalid requests needing prompt fix: attach the validation error from the last attempt as context.
- If chunking documents: assign **multiple custom IDs** per original document (one per chunk).
- Always **upsert** results by custom ID — never duplicate.

---

### Example failure: swapped fields

Landlord lease extraction: `subtenant` and `tenant` fields swapped.  
Batch job succeeded (structurally valid) but semantically wrong.  
Fix: test sample with synchronous API → fix layout instruction → rerun.  
Had this gone to full corpus first: expensive re-run of thousands of documents.

---

### Exam pattern

```
Design prompt → test on sample (synchronous) → iterate
  → full corpus (Batch API)
    → for failed items: investigate 5 examples, fix, resubmit only those
```
