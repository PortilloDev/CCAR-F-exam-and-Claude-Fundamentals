# Lecture 65: Skills & Commands: CLAUDE.md vs. Skill vs. Hook

**Section:** Claude Code in a Team  
**Lecture ID:** 58217067

---

## Transcript

### Decision tree

```
Should this rule NEVER be skipped?
├── YES → Hook (settings.json shell script via pre_tool_use)
└── NO  → Should it always apply across all projects?
           ├── YES → CLAUDE.md (project or global)
           └── NO  → Is it task-based (debugging vs. developing vs. security review)?
                      └── YES → Skill (task-based, on-demand)
```

---

### Comparison table

| | CLAUDE.md | Skill | Hook |
|--|-----------|-------|------|
| **When loaded** | Always (or on dir match) | Description always; body on demand | Always on pattern match |
| **Enforcement** | Probabilistic | Probabilistic | **Deterministic** |
| **Best for** | Standing conventions | Task-specific procedures | Zero-exception business rules |
| **Token efficiency** | Low (all at startup) | High (body on demand) | N/A |

---

### Symptom → fix examples

**Symptom 1:** 400-line CLAUDE.md with coding standards + PR review + deployment + migrations.  
**Fix:** Keep coding standards in CLAUDE.md. Move PR review, deployment, migration → separate skills (loaded on demand).

**Symptom 2:** Endpoint documentation in CLAUDE.md causes context overflow when debugging.  
**Fix:** Move endpoint docs → a skill with a development-specific trigger. Path-specific rules won't work — they'd load during debugging too (same file type).

**Symptom 3:** Need to prevent a `psql` command from running against production.
- CLAUDE.md rule: probabilistic — will eventually be ignored.
- Skill: probabilistic — might not trigger or get loaded.
- **Hook** (`pre_tool_use` on Bash, filter for production URL): **deterministic** — always fires.

---

### Three rules to memorize

1. **Never skippable** → `settings.json` hook (shell script).
2. **Always on, all projects** → CLAUDE.md (ideally in Git).
3. **Task-specific procedure** → Skill (name + description in frontmatter).
