# Lecture 64: Skills & Commands: Slash Commands

**Section:** Claude Code in a Team  
**Lecture ID:** 58217061

---

## Transcript

### Two ways to create slash commands

**Option 1 — Commands folder (older, still exam-relevant):**
```
.claude/commands/
  audit.md        ← invoked as /audit [arguments]
```

```markdown
<!-- audit.md -->
Audit the following code for security issues: $ARGUMENTS
```

**Option 2 — Skills folder (modern, preferred):**
```
.claude/skills/
  audit/
    SKILL.md      ← also creates /audit slash command
```

Both create `/audit`. The skills approach is more powerful (supports reference files, context fork, etc.).

**Exam note:** The exam still emphasizes `.claude/commands/` — know both approaches.

---

### Sharing vs. personal commands

| Location | Shared with team? | How |
|----------|-----------------|-----|
| `<repo>/.claude/commands/` | ✅ Yes | Git — every pull brings updates |
| `~/.claude/commands/` | ❌ No | Personal only |

---

### Naming collision problem

If you have a personal `/audit` and your team also has a `/audit`, they will collide at the slash command level. You may accidentally use your personal version while teammates get the team version — and you miss team updates.

**Fix:** Prefix personal commands with your name:
```
.claude/commands/
  audit.md           ← team's version
  ivan-audit.md      ← your personal version → /ivan-audit
```

**Cannot do:**
- No `override` key to specify which takes priority.
- No username-based conditional loading.
- Personal copies of team skills won't receive team updates.

**Rule:** Separate team and personal commands by name from the start.
