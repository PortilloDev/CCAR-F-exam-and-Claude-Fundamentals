# Lecture 63: Skills & Commands: Skills as Procedures

**Section:** Claude Code in a Team  
**Lecture ID:** 58217055

---

## Transcript

### Skill file structure

```
.claude/skills/
  verify/
    SKILL.md          ← main skill file (frontmatter + procedure)
    checklist.md      ← optional reference file
    examples/         ← optional supporting files
```

```markdown
---
name: verify
description: >
  Check that a change is really shippable. Use after editing source,
  when asked whether the work is done, and before opening a PR.
allowed-tools: ["Read", "Bash"]
argument-hint: "<path-to-changed-files>"
context: fork
---

## Verification procedure

1. Run `npm test` and confirm all tests pass.
2. Check test coverage hasn't dropped below 80%.
3. Confirm no new `console.log` statements in production code.
4. Verify no hardcoded credentials.
```

---

### How skills load

| Part | When loaded |
|------|------------|
| Frontmatter (name + description) | **Always** — at conversation start |
| Body (procedure steps) | **On demand** — when Claude matches the task to the skill |
| Reference files in the skill folder | On demand — if the skill needs them |

This is the key advantage over CLAUDE.md imports: the body is only loaded when needed.

---

### Frontmatter fields

| Field | Purpose |
|-------|---------|
| `name` | The slash command name (`/verify`) |
| `description` | Trigger condition — what task/moment should load this skill |
| `allowed-tools` | Pre-approved tools for this skill (also restricts to this list) |
| `argument-hint` | Prompt shown when invoking via slash command — guides the argument |
| `context: fork` | Runs the skill in a **separate side session** — result returned to main session |

---

### `context: fork` — when and why

**Without fork:** Analysis skill runs inside the main conversation → intermediate ideas pollute context → model may drift from original task.

**With fork:**
1. Main session continues.
2. Skill spins up a fresh isolated context.
3. Skill does its work (e.g., code analysis, brainstorming).
4. Only the **result** comes back to the main session.

**Use fork for:** analysis, brainstorming, verification — any side work where intermediate steps should not bleed into the main conversation.

**Non-solutions** (don't work):
- Cheaper model — still consumes tokens in that model's context.
- Smaller skills split into three — still fills context if all three are needed.
- Summarize at the end — still needs the full run first.
