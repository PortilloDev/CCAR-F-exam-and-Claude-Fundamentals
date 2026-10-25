# Lecture 53: Intro: Decomposition and Sessions

**Section:** Agentic Architecture  
**Lecture ID:** 58006591

---

## Transcript

A saved session can help you continue a code investigation, but it also brings back file contents and tool results from earlier work. If colleagues have changed those files, some evidence and conclusions may need checking. Restarting everything is not always necessary — but neither is blindly resuming.

### What this chapter covers

- **Task decomposition:** breaking a task into smaller pieces.
- **Fixed pipeline** — for predictable, known workflows.
- **Adaptive plan** — for open-ended tasks where the next step depends on findings.
- **Session resumption** — how to continue and how to signal changed files for targeted re-analysis.
- **When to start fresh** — when stale information makes resumption counterproductive.
- **Session forking** — creating independent explorations from a shared analysis baseline to compare approaches.

**Key constraint:** A fork baseline must still be valid — if the baseline is stale, forking inherits the problem.

**Goal:** Choose the right task structure and decide whether to resume, fork, or restart a session based on the validity of existing context.
