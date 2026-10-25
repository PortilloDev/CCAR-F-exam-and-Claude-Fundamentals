# Lecture 58: Configuration: CLAUDE.md

**Section:** Claude Code in a Team  
**Lecture ID:** 58217039

---

## Transcript

### CLAUDE.md file hierarchy

| Level | Location | Shared? | Loaded when? |
|-------|----------|---------|-------------|
| **Managed policy** | Anthropic-enforced | N/A | Always (cannot override) |
| **User** | `~/.claude/CLAUDE.md` | No (personal) | Always, every project |
| **Project** | `<repo>/CLAUDE.md` or `<repo>/.claude/CLAUDE.md` | Yes (Git) | Always, when in this project |
| **Directory** | `<repo>/src/api/CLAUDE.md` | Yes (Git) | On demand when agent works in that folder |
| **Local** | `<repo>/CLAUDE.local.md` | No (gitignored) | Always, personal per-project overrides |

---

### How context is built at conversation start

1. Managed policy files (Anthropic).
2. User global CLAUDE.md.
3. Project-level CLAUDE.md (one or more).
4. Directory-specific CLAUDE.md files — loaded **on demand** as the agent works in those folders.

---

### The teammate onboarding problem

If a senior developer stores key rules (e.g., "always include comprehensive error handling") in their **personal** user CLAUDE.md, new teammates will never see those rules — their code will lack those patterns.

**Fix:** Periodically promote important rules from personal user files → the shared project CLAUDE.md in Git.

---

### Debugging which files are loaded

```
/memory
```

Running `/memory` inside Claude Code shows all CLAUDE.md files loaded in the current session. Use this to verify configuration and identify rules that exist only in personal files.
