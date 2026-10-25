# Lecture 92: Scale & Review Architecture: Multi-Pass Review

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217259

---

## Transcript

### The attention dilution problem

Reviewing 10 documents in a single context → attention per document is low → errors missed.  
Reviewing each document in its own pass → full attention → precise findings.

---

### Two-pass architecture

**Pass 1: Local (per-file) pass**
- Review each document independently (parallel — files don't depend on each other).
- Find: all issues within the file (code bugs, structure errors, local inconsistencies).
- Also detect: what the file exports/imports (integration points).

**Pass 2: Integration pass**
- Review cross-file dependencies using findings from Pass 1.
- Find: data flow errors between classes, cross-reference inconsistencies, import/export mismatches.

---

### What each pass detects

| Finding type | Pass |
|-------------|------|
| Bug in a loop | Local (Pass 1) |
| Data flow: price sent as cents, displayed as dollars | Integration (Pass 2) |
| Missing import | Local (Pass 1) |
| Interface mismatch between two modules | Integration (Pass 2) |

---

### Order is mandatory

Pass 1 → Pass 2. Cannot reverse:
- Pass 2 needs integration points identified in Pass 1.
- Running Pass 2 first would require loading all files at once → attention dilution.

**Parallelism:** Pass 1 runs all per-file reviews in parallel (files are independent of each other).

---

### Exam pattern

```
Large codebase / document set review:
  └── Pass 1: per-file, independent, parallel
        └── Pass 2: integration, uses Pass 1 findings, serial
```
