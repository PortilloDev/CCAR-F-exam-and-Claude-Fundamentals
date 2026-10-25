# Lecture 76: Intro: Precision

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58006603

---

## Transcript

A code-review assistant is useful only if its findings deserve attention. When it repeatedly flags harmless style differences as serious problems, developers may stop reading its comments and miss a real defect. Improving the prompt starts with explaining what counts as a useful finding and how severity should be assigned.

### What this chapter covers

- **Explicit criteria:** defining what the reviewer should flag, what it should ignore, and how to assign severity — comparing explicit criteria with vague instructions like "be conservative."
- **Precision vs. coverage:** disabling categories that produce too many false positives while improving their criteria offline; doing two things well beats five things poorly.
- **Few-shot examples:** choosing examples that show the boundary between an acceptable result and a problem; explaining the reasoning behind each answer; using examples to establish output format.
- **Document extraction:** examples for different layouts; representing information that the source document does not contain.

**Goal:** Write clearer review criteria and select examples that address a specific source of inconsistency.
