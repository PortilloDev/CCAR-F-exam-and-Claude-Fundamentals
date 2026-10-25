# Lecture 89: Intro: Scale and Review Architecture

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58006607

---

## Transcript

As the amount of work grows, you need to decide how to schedule it and how to check the results. A large extraction job may tolerate delayed responses, while a decision needed immediately requires a synchronous request. The review process also needs enough attention for each part of the work.

### What this chapter covers

- **Message Batch API:** independent batch requests, custom identifiers, polling for completion, one model response per request.
- **Batch vs. synchronous:** choosing by deadline and submission frequency.
- **Multi-pass review:** reviewing files individually before an integration pass; examining both local issues and cross-file dependencies.
- **Multi-instance review:** independent reviewer with no generator session history; reconciling findings without majority voting.
- **Test small & batch corpus:** testing extraction prompt on a sample first, then modifying and resubmitting only failed items using their custom IDs.

**Goal:** Choose a processing approach, recover failed work, and organize a review that preserves useful findings.
