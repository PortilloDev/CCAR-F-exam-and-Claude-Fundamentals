# Lecture 60: Configuration: Splitting CLAUDE.md

**Section:** Claude Code in a Team  
**Lecture ID:** 58217045

---

## Transcript

### The problem: monolithic CLAUDE.md

A 500-line file with TypeScript conventions, testing standards, API patterns, and deployment procedures becomes hard to maintain — rules get duplicated, redundant rules go unnoticed, new rules get appended at the end.

---

### Import syntax (exam term: `@`)

```markdown
<!-- In CLAUDE.md -->
Our coding standards live at @.claude/standards/typescript.md
@.claude/standards/testing.md
```

- The `@` symbol imports another file.
- The text before the `@` is optional label.
- **All imported files are loaded at conversation start** — no token savings, just organizational clarity.

---

### Loading behaviors comparison

| Approach | Loaded when? | Token savings? |
|----------|-------------|---------------|
| Single CLAUDE.md with import `@` | All at startup | ❌ No |
| Folder-level CLAUDE.md files | On demand (when agent works in folder) | ✅ Yes |
| Skills (SKILL.md) | Description at startup; body on demand | ✅ Yes |
| Path-specific glob rules | On demand (when file path matches) | ✅ Yes |

---

### What splitting DOES fix

- Easier navigation between topics.
- Cleaner code reviews (changes are in specific files).
- Shared sub-files referenced by multiple projects (e.g., a shared testing standard).

### What splitting does NOT fix

- Bad rule wording — a well-organized file of vague rules is still vague.
- Token usage — `@` imports load everything at startup.
- Enforcement — for deterministic rules, use **hooks** (not CLAUDE.md).

---

### Hybrid approach

```
.claude/
  CLAUDE.md          ← global project rules (always loaded)
  standards/
    typescript.md    ← imported via @
  skills/
    pr-review/       ← loaded on demand via skill description
src/
  api/
    CLAUDE.md        ← loaded on demand when working in src/api/
```
