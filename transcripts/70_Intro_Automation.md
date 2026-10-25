# Lecture 70: Intro: Automation

**Section:** Claude Code in a Team  
**Lecture ID:** 58006601

---

## Transcript

When Claude Code runs in an automated pipeline, the interaction model changes: there is no developer to answer approval prompts, and output must be machine-readable rather than conversational.

### What this chapter covers

- **Headless mode:** running Claude Code as a subprocess with `-p`, structured JSON output, permission modes that skip approval prompts, tool restrictions, and turn limits.
- **GitHub Action integration:** triggering Claude Code from CI workflows, responding to PR comments, posting inline review findings back to the pull request.
- **Repeating reviews:** using CLAUDE.md as the review standard, running the reviewer in a fresh independent session to avoid anchoring bias, and feeding previous findings as context to avoid re-reporting resolved issues.
- **Testing standards:** supplying existing test suites and telling Claude where fixtures and test-runner commands live so generated tests are consistent with the project's conventions.

**Goal:** Run Claude Code without human supervision, produce structured output downstream systems can consume, and integrate it cleanly into GitHub workflows.
