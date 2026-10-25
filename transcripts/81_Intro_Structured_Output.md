# Lecture 81: Intro: Structured Output

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58006605

---

## Transcript

Suppose Claude extracts an invoice into a record that passes schema checks, but the total differs from the amount printed on the document. The record has the expected fields and types, yet one value is wrong. Schema validity is not semantic correctness.

### What this chapter covers

- **Documents to JSON:** defining an extraction tool, forcing its call, and checking facts that a valid schema does not establish.
- **Missing fields:** nullable fields for absent data, "unclear"/"other" enum values, prompt-level normalization instructions.
- **Schema validation:** structural errors (wrong type, misplaced field) vs. semantic errors (totals don't add up) — when to retry vs. when to send to human review.
- **Grounded confidence score:** per-field scoring calibrated against labeled data; routing decisions based on score vs. threshold.
- **Learning from dismissed findings:** collecting false positives, detecting patterns, improving criteria and few-shot prompts.

**Goal:** Design an extraction process that checks values against their source and directs uncertain cases to a human reviewer.
