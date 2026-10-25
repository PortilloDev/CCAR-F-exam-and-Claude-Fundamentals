# Lecture 78: Precision: Precision vs. Coverage

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217147

---

## Transcript

### The core principle

**Do two things well instead of five things poorly.**

In CI/CD pipelines and document processing pipelines, false positives immediately destroy trust. A developer who sees noisy results will ignore the whole report — including the high-precision findings.

---

### Example: code review false-positive rates

| Category | False-positive rate |
|----------|-------------------|
| Style & naming | 52% |
| Documentation | High |
| Performance | Moderate |
| Security | Low (accurate) |

**Problem:** a developer reads style findings first, sees noise, and ignores the security findings.

---

### The fix: disable noisy categories

1. **Disable** noisy categories (style, naming, documentation).
2. **Keep** high-precision categories (security).
3. **Fix** disabled categories offline with few-shot prompts.
4. **Re-enable** once performance improves.

---

### Why the alternatives don't work

| Alternative | Why it fails |
|-------------|-------------|
| Add a confidence score to noisy results | Low-confidence results are still visible; trust still erodes |
| Reduce strictness (fewer results overall) | Also loses real positives from high-precision categories |
| Keep noisy categories | Trust erodes; good findings get ignored |

---

### Correct sequence

```
Measure false-positive rate per category
└── Category is noisy? → Disable it
    └── Fix offline with few-shot prompts
        └── Re-enable once performance is acceptable
```

**Key exam point:** reducing strictness hurts precision categories (you miss real security findings). Disabling noisy categories preserves precision in the categories that work.
