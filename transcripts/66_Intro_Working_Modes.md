# Lecture 66: Intro: Working Modes

**Section:** Claude Code in a Team  
**Lecture ID:** 58006597

---

## Transcript

Different development tasks need different working modes. A broad system change benefits from planning and exploration before any edits. A small, well-defined correction may go straight to execution. Permission settings also differ between interactive work and unattended pipelines.

### What this chapter covers

- **Plan mode vs. direct execution:** choosing based on scope, need for exploration, and alternative approaches.
- **Permission choices:** interactive vs. unattended (headless) — avoiding pipelines that wait for approval.
- **Iterative refinement techniques:**
  - Interview before implementation to uncover requirements.
  - Input/expected-output examples to clarify behavior.
  - Write tests first — guide changes through failing tests.
- **Batching vs. sequencing:** fixes that affect each other go in one message; independent fixes can be handled one at a time.

**Goal:** Choose how Claude should approach a task, set suitable approval behavior, and provide feedback that resolves the actual problem.
