# Study Doc — Section 5: Prompt Engineering and Structured Output

**Domain:** Prompt Engineering and Structured Output  
**Files:** 76–95

---

## Precision — Explicit Criteria

**Problem:** vague adjectives ("be conservative", "check accuracy") → non-reproducible results.  
**Fix:** categorical criteria.

```
Flag a comment only if it:
- Contradicts the code
- Names something that does not exist
- Points at a bug already fixed

Do NOT flag:
- Stylistic differences
- Minor wording inaccuracies
- Missing comments (absence)
```

Severity calibration: add one few-shot example per severity level (Critical/High/Low) as a bar to compare against.

**EXAM TRAP:** "Be conservative" without examples = uncalibrated = anti-pattern.  
**EXAM TRAP:** Self-assigned confidence scores without labeled data = anti-pattern.

---

## Precision vs. Coverage

**Do two things well instead of five things poorly.**

If a category has >20% false-positive rate:
1. **Disable** it from the pipeline
2. **Fix** offline with few-shot prompts
3. **Re-enable** once performance improves

**EXAM TRAP:** Reducing strictness (fewer results) also removes real positives from high-precision categories.  
**EXAM TRAP:** Adding a confidence score to noisy results doesn't rebuild trust.

---

## Few-Shot Examples — When and How

Use ONLY for ambiguous boundary cases that survive criteria improvements.

Decision flow:
```
Tool descriptions well-defined?  NO → fix descriptions first
Categorical criteria in place?   NO → add criteria first
Still false positives at boundary? YES → add few-shot examples for that boundary only
```

One or two examples per boundary is enough — Claude generalizes.

**Evidence the prompt works:** false positives drop on cases NOT in the few-shot set (generalization).  
**Not evidence:** only performs better on the exact examples provided.

---

## Document Extraction — Missing Fields

Two failure modes:
| Mode | Cause |
|------|-------|
| Claude **invents** a value | Field required but absent; retry pressure increases hallucination |
| Claude returns **empty** but value was present | Layout not understood |

**EXAM TRAP:** Recommending a retry when the value is absent from the document → increases hallucination, not accuracy.

When field is missing — fix options:
- Use `null` type: `"type": ["string", "null"]`
- Add enum escape values: `"unclear"`, `"other"`
- Remove `required` if truly optional

---

## Structured Output — The Forced Tool Call Pattern (EXAM PREFERRED)

```python
tools = [{"name": "extract_invoice", "input_schema": schema}]
response = client.messages.create(
    ...,
    tools=tools,
    tool_choice={"type": "tool", "name": "extract_invoice"}
)
extracted = response.content[0].input  # structured output — no prose
```

**Key schema parameters:**
- `additionalProperties: false` — prevents extra fields
- `strict: true` — stops flow if required field missing
- `tool_choice` forced — guarantees exactly this tool is called

**EXAM TRAP:** Plain prompt asking for JSON → prose response (unreliable). Always use forced tool call.  
**EXAM TRAP:** Schema validates structure — NOT semantic correctness. Add deterministic code for arithmetic checks.

---

## Schema Validation — Retry vs. Human Review

Retry helps:
- Date in wrong format
- Value in wrong field
- Arithmetic error (single digit)

Retry does NOT help:
- Value not in the document at all → send to **human review**

---

## Grounded Confidence Score

Anti-patterns:
- Pure self-rating (no calibration data)
- Single score for whole document
- Threshold set by intuition

Correct pattern:
1. Score **per field** (not per document)
2. Calibration dataset (labeled few-shot examples)
3. Threshold derived from data
4. Route: above threshold → automate; below → human review

**Routing matrix:**

| Source contradicts itself | Confidence | Action |
|--------------------------|------------|--------|
| No | Above threshold | Automate |
| Yes | High | Human review |
| Either | Below threshold | Human review |

---

## Message Batch API

| Dimension | Batch API | Synchronous |
|-----------|-----------|-------------|
| Cost | Up to 50% cheaper | Full price |
| Latency | Up to 24h | Seconds |
| Tool calls | Single forced tool only | Full multi-turn |
| Best for | Large volume, flexible SLA | Tight deadlines, interactive |

**EXAM TRAP:** Batch cannot do multi-turn. It can do ONE forced tool call only.  
**EXAM TRAP:** 24h is the max SLA — plan your time budget accordingly (preparation + batch + downstream).

---

## Multi-Pass Review

| Pass | What it finds |
|------|--------------|
| Pass 1 (per-file, parallel) | Bugs in code, local issues, integration points |
| Pass 2 (integration, serial) | Cross-file data flow errors, import/export mismatches |

Order is mandatory: Pass 1 → Pass 2.

---

## Multi-Instance Review

Fresh session for every review. Do NOT pass:
- Generating session's chat history
- Any hint "the code is mostly fine"
- Generator's reasoning trace

Reconcile findings: **union** (not majority vote). Keep all unique findings. For duplicates: keep higher severity.

**EXAM TRAP:** Majority voting cuts real positives — unacceptable in security contexts.

---

## Test Small → Batch Corpus

1. Build representative sample (one example per document type, longest docs, all layouts)
2. Optimize prompt on sample using **synchronous API** (fast feedback)
3. Submit full corpus to **Batch API** only after one clean pass

Resubmission: investigate only the failed items (never rerun full job). Track by custom ID. Upsert — never duplicate.

---

## Learning From Dismissed Findings

Add `detected_pattern` field to schema → Claude categorizes dismissals → group by pattern → build "do not flag" blocks + few-shot examples → watch for generalization.
