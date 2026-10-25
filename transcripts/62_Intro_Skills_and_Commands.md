# Lecture 62: Intro: Skills and Commands

**Section:** Claude Code in a Team  
**Lecture ID:** 58006595

---

## Transcript

Repeated tasks (e.g., checking a release) are easier to manage when the procedure is stored with the project. If everyone types steps from memory, one person may omit a test or use different review criteria.

**Skills** and **slash commands** package instructions for reuse.

### What this chapter covers

- Skill file frontmatter: description (for discovery), argument hints, tool approval, isolated context.
- How Claude discovers a skill automatically vs. explicit invocation via slash command.
- Sharing a slash command through version control; creating a personal variant with a distinct name.
- Comparing **Skills** vs. **CLAUDE.md** vs. **Hooks**:
  - Task-specific procedure → Skill
  - Ongoing convention → CLAUDE.md
  - Rule that code must enforce → Hook

**Goal:** Package a reusable procedure and put each kind of instruction in the appropriate place.
