# Lecture 96: Intro: Context as a Budget

**Section:** Context and Reliability  
**Lecture ID:** 58006609

---

## Transcript

During a long code investigation, a conversation can accumulate file contents, test results and decisions that matter later. Claude can only work with a finite amount of context at once, so you need a plan for keeping essential information available. Summarising can reduce the volume, but it may also remove an identifier, an amount or an instruction that you still need.

### What this chapter covers

- **Context budget:** deciding which facts must remain exact and storing them separately from summaries; trimming tool output; placing key findings first.
- **Directed compaction:** retaining important details through the compact step; reloading omitted facts from saved state.
- **Verbose discovery & degradation:** exploring a codebase through search before reading full files; using subagents to keep verbose exploration out of the main conversation.
- **Recovering state:** scratchpads, saved agent findings, coordinator's record of subagent locations — supporting recovery after crashes or context overflow.

**Goal:** Choose what stays in context, preserve exact identifiers and amounts needed for decisions, and restore context for a long task from files saved on disk.
