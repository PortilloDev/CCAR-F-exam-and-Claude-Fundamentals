# Lecture 77: Precision: Explicit Criteria

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217133

---

## Transcript

### The problem with vague adjectives

Instruction: "check that comments are accurate and up to date"

**Result:** false positives (TODO markers flagged), real issues missed (comment contradicts return type).

**Root cause:** adjectives like "accurate" are interpreted differently each run — results are not reproducible.

---

### Categorical criteria (the fix)

```
Flag a comment only if it:
- Describes behaviour that contradicts the code
- Names a function or variable that does not exist
- Points at a bug that has already been fixed (TODO/FIXME)

Do NOT flag:
- Comments that are stylistically outdated
- Comments with minor wording inaccuracies
- Missing comments (absence)
```

**Effect:** measurable, reproducible. Claude knows exactly what to look for and what to skip.

---

### Adding severity with few-shot examples

When adding severity levels, define a calibration example per level:

| Severity | Example |
|----------|---------|
| Critical | Null pointer exception during payment processing |
| High | SQL injection vulnerability |
| Low | Slow algorithm over a small list |

**Why examples:** "be conservative" is uncalibrated. Examples give Claude a concrete bar to compare against.

---

### Anti-patterns to avoid

| Anti-pattern | Problem |
|-------------|---------|
| "be conservative" | Uncalibrated — varies per run |
| "report high-confidence findings" (no examples) | Self-confidence has no baseline |
| Self-assigned confidence scores | Not calibrated against labeled data |

---

### Decision flow

```
Does the instruction use vague adjectives?
└── YES → Replace with categorical criteria (flag only if X, do not flag Y)
           └── Are there ambiguous severity levels?
               └── YES → Add few-shot examples per severity level
```
