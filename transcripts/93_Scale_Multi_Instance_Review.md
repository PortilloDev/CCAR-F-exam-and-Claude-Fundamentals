# Lecture 93: Scale & Review Architecture: Multi-Instance Review

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217263

---

## Transcript

### The bias problem with self-review

If Claude generated the code and you ask the **same session** to review it:
- The long reasoning trace that produced the code is still in context.
- Claude anchored on "this is correct" during generation.
- Review in the same session will miss real bugs (anchoring bias).

Increasing effort (`extended_thinking`, self-critique instructions) helps slightly but not enough.

---

### Fresh reviewer session (correct approach)

Start a **completely new session** for every review. Pass only:
- The code/document to review (no commentary, no hints).
- Project context: CLAUDE.md, skills, conventions.
- Explicit review criteria.

**Do NOT pass:**
- The generating session's chat history.
- Any resumed or forked generating session.
- Any reasoning about how the code was produced.
- Any hint that "the code is mostly fine."

---

### Reconciling findings from multiple reviewers

**Do NOT use majority voting:**

> Reviewer A: 2 bugs. Reviewer B: 1 bug. Reviewer C: 1 bug.  
> Majority vote → 1 bug wins. But Reviewer A was the most precise one.

**Use union instead:**

| Situation | Action |
|-----------|--------|
| Finding appears in only one reviewer | Keep it |
| Finding appears in multiple reviewers (duplicate) | Keep one instance |
| Same finding, different severities | Keep the **higher** severity |

Majority voting cuts false positives but also cuts real positives — unacceptable in security contexts.

---

### Confidence score in multi-instance review

Apply per-finding confidence scores at each pass, calibrated against labeled data — same pattern as document extraction but per finding instead of per field.
